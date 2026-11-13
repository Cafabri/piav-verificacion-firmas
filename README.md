# Verificación de firmas manuscritas offline

Práctica de **Procesamiento de Imágenes, Audio y Vídeo (PIAV)**, Grado en Ciencia e Ingeniería de Datos.

**Sistema paramétrico con OpenCV** sobre la base de datos **CEDAR**: preprocesado (mediana, Otsu, limpieza, recorte), 13 características geométricas (densidad de tinta, relación ancho/alto, centroide, componentes conexas, momentos de Hu, tinta por cuadrantes, inclinación) y una puntuación basada en la distancia z al patrón del usuario, con un umbral global de decisión fijado en desarrollo.

## Hipótesis

### Hipótesis 1: sistema paramétrico (OpenCV)

**Planteamiento.** De cada firma sacamos estas medidas: la densidad de tinta, la proporción entre ancho y alto, la posición del centro de la firma (horizontal y vertical), el número de trazos sueltos (componentes conexas), tres momentos de Hu que describen la forma global, el reparto de la tinta en los cuatro cuadrantes y la inclinación. En total son 13 valores por firma. Con las 10 firmas de referencia vemos cómo suelen ser esas medidas para cada persona. Si una firma nueva se parece lo suficiente, la aceptamos.

**Qué esperamos.**
- Que funcione, pero con bastante error: un **EER de entre el 15 % y el 25 %**.
- Que le cueste distinguir las **falsificaciones hábiles**. Quien falsifica imita bien el aspecto general de la firma, y ese aspecto general es lo único que miden nuestras características. Por eso muchas falsificaciones van a parecer genuinas para el sistema.
- Que, al usar el mismo umbral para todos, la **FAR y la FRR salgan algo descompensadas** en evaluación, ya que no todo el mundo firma igual de regular.

**Por qué.** Lo que de verdad delata a un falsificador está en los detalles del trazo: va más lento, le tiembla la mano y la línea no tiene el mismo grosor. Nuestras medidas no recogen nada de eso. Además, 10 firmas son pocas para saber cuánto varía cada persona al firmar.

**Cómo lo comprobamos.** Con el EER y la curva DET de los usuarios de evaluación, y con la FAR y la FRR que salen usando el umbral fijado en desarrollo.

### Hipótesis 2: sistema de aprendizaje automático

**Planteamiento.** En vez de elegir nosotros las medidas, una red neuronal aprende sola a describir las firmas a partir de los usuarios de desarrollo. Después comparamos cada firma con las 10 de referencia de su usuario.

**Qué esperamos.**
- Que **funcione mejor que el paramétrico**, con un EER más bajo.
- Que, al tener pocos usuarios para entrenar, la red **se aprenda demasiado bien a esos usuarios** y funcione peor con los nuevos.
- Que con el hiperparámetro (por ejemplo, las épocas) el error **baje al principio y luego se estanque o empeore**, de modo que el mejor valor sea uno intermedio.

**Por qué.** La red puede fijarse en detalles del trazo que un falsificador no consigue imitar y que nosotros no sabríamos medir a mano.

**Cómo lo comprobamos.** Comparando las curvas DET de los dos sistemas en la misma gráfica y viendo cómo cambia el error con al menos 3 valores del hiperparámetro.
