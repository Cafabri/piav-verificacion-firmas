# Verificación de firmas manuscritas offline

Práctica de **Procesamiento de Imágenes, Audio y Vídeo (PIAV)**, Grado en Ciencia e Ingeniería de Datos.

**Sistema paramétrico con OpenCV** sobre la base de datos **CEDAR**: preprocesado sencillo (filtro de mediana, Otsu y recorte), 5 características (densidad de tinta, relación ancho/alto, centro de la firma en horizontal y en vertical, y componentes conexas) y una puntuación basada en la distancia de cada firma a la firma media del usuario, con un umbral global fijado en desarrollo.

## Hipótesis

### Hipótesis 1: sistema paramétrico (OpenCV)

**Planteamiento.** Primero limpiamos cada firma (filtro de mediana, binarización con Otsu y recorte) y después sacamos 5 medidas sencillas: la densidad de tinta, la proporción entre ancho y alto, la posición del centro de la firma en horizontal y en vertical, y el número de trazos separados (componentes conexas). Con las 10 firmas de referencia calculamos la firma "media" de cada persona. Para comparar una firma nueva, ponemos todas las medidas en la misma escala y calculamos lo lejos que está de esa firma media: cuanto más cerca, más se parece.

**Umbral.** Usaremos un único umbral global, el mismo para todos los usuarios. Lo fijaremos con los 30 usuarios de desarrollo en el punto en el que la FAR y la FRR son iguales (el punto del EER). Para encontrarlo, puntuaremos todas las firmas de prueba de los usuarios de desarrollo, calcularemos la FAR y la FRR para cada umbral posible y nos quedaremos con el valor en el que ambas coinciden. Después no lo tocaremos, ni en evaluación ni con los usuarios nuevos de la prueba final. Si una firma tiene una puntuación igual o mayor que el umbral, la aceptamos; si no, la rechazamos.

**Qué esperamos.**
- Que funcione, pero con bastante error: un **EER de entre el 20 % y el 30 %**.
- Que le cueste distinguir las **falsificaciones hábiles**. Quien falsifica imita bien el aspecto general de la firma, y ese aspecto general es lo único que miden nuestras características. Por eso muchas falsificaciones van a parecer genuinas para el sistema.
- Que, al usar el mismo umbral para todos, la **FAR y la FRR salgan algo descompensadas** en evaluación, ya que no todo el mundo firma igual de regular.

**Por qué.** Lo que de verdad delata a un falsificador está en los detalles del trazo: va más lento, le tiembla la mano y la línea no tiene el mismo grosor. Nuestras medidas no recogen nada de eso. Además, con solo 5 medidas generales, firmas bastante distintas pueden dar valores parecidos.

**Cómo lo comprobamos.** Con el EER y la curva DET de los usuarios de evaluación, y con la FAR y la FRR que salen usando el umbral fijado en desarrollo.

### Hipótesis 2: sistema de aprendizaje automático

**Artículo.** Vamos a intentar replicar una versión simplificada de SigNet:

> Hafemann, L. G., Sabourin, R., & Oliveira, L. S. (2017). Learning features for offline handwritten signature verification using deep convolutional neural networks. *Pattern Recognition, 70*, 163–176. https://doi.org/10.1016/j.patcog.2017.05.012

En el artículo entrenan una red neuronal convolucional para que aprenda a reconocer quién ha hecho cada firma. Una vez entrenada, le quitan la última capa y la usan para convertir cualquier firma en un vector de características, incluso de personas que la red no ha visto nunca.

**Planteamiento.** En vez de entrenar una red desde cero como en el artículo, partiremos de una red ya preentrenada (ResNet18) y la ajustaremos con los usuarios de desarrollo para que aprenda a distinguir quién firma. Después la usaremos para sacar el vector de características de cada firma. Para decidir, en lugar de entrenar un clasificador por usuario como hace el artículo, compararemos cada firma con sus 10 de referencia mediante la similitud coseno. El hiperparámetro que estudiaremos es el número de épocas de entrenamiento.

**Umbral.** Igual que en el paramétrico: un único umbral global, fijado con usuarios de desarrollo en el punto del EER y sin cambiarlo después.

**Qué esperamos.**
- Que **funcione mejor que el paramétrico**, con un EER más bajo.
- Que, al tener pocos usuarios para entrenar, la red **se aprenda demasiado bien a esos usuarios** y funcione peor con los nuevos.
- Que, al aumentar el número de épocas, el error **baje al principio y luego se estanque o empeore**, de modo que el mejor valor sea uno intermedio.

**Por qué.** La red puede fijarse en detalles del trazo que un falsificador no consigue imitar y que nosotros no sabríamos medir a mano.

**Cómo lo comprobamos.** Comparando las curvas DET de los dos sistemas en la misma gráfica y viendo cómo cambia el error con al menos 3 valores del hiperparámetro.
