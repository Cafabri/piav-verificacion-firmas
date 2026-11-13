# Verificación de firmas manuscritas offline

Práctica de **Procesamiento de Imágenes, Audio y Vídeo (PIAV)**, Grado en Ciencia e Ingeniería de Datos.

**Sistema paramétrico con OpenCV** sobre la base de datos **CEDAR**: preprocesado (mediana, Otsu, limpieza, recorte), 13 características geométricas (densidad de tinta, relación ancho/alto, centroide, componentes conexas, momentos de Hu, tinta por cuadrantes, inclinación) y una puntuación basada en la distancia z al patrón del usuario, con un umbral global de decisión fijado en desarrollo.

## Hipótesis

**Planteamiento.** De cada firma sacamos estas medidas: la densidad de tinta, la proporción entre ancho y alto, la posición del centro de la firma (horizontal y vertical), el número de trazos sueltos (componentes conexas), tres momentos de Hu que describen la forma global, el reparto de la tinta en los cuatro cuadrantes y la inclinación. En total son 13 valores por firma. Con las 10 firmas de referencia vemos cómo suelen ser esas medidas para cada persona. Si una firma nueva se parece lo suficiente, la aceptamos.

**Qué esperamos.**
- Que funcione, pero con bastante error: un **EER de entre el 15 % y el 25 %**.
- Que le cueste distinguir las **falsificaciones hábiles**. Quien falsifica imita bien el aspecto general de la firma, y ese aspecto general es lo único que miden nuestras características. Por eso muchas falsificaciones van a parecer genuinas para el sistema.
- Que, al usar el mismo umbral para todos, la **FAR y la FRR salgan algo descompensadas** en evaluación, ya que no todo el mundo firma igual de regular.

**Por qué.** Lo que de verdad delata a un falsificador está en los detalles del trazo: va más lento, le tiembla la mano y la línea no tiene el mismo grosor. Nuestras medidas no recogen nada de eso. Además, 10 firmas son pocas para saber cuánto varía cada persona al firmar.

**Cómo lo comprobamos.** Con el EER y la curva DET de los usuarios de evaluación, y con la FAR y la FRR que salen usando el umbral fijado en desarrollo.

## Resultados (usuarios de evaluación 31–43)

| Sistema | EER | FAR (umbral fijo) | FRR (umbral fijo) |
|---|---|---|---|
| Paramétrico OpenCV | 16,0 % | 12,2 % | 18,7 % |

## Estructura

| Archivo | Contenido |
|---|---|
| `practica_firmas.ipynb` | Notebook: datos y protocolo, funciones de evaluación biométrica y sistema paramétrico |

## Ejecución

1. Descargar la base de datos del Campus Virtual y descomprimirla así:
   ```
   data/genuinas/full_org/full_org/original_<usuario>_<n>.png
   data/falsificadas/full_forg/full_forg/forgeries_<usuario>_<n>.png
   ```
2. Instalar las dependencias:
   ```
   pip install opencv-python numpy pandas scipy matplotlib
   ```
3. Ejecutar `practica_firmas.ipynb` de principio a fin.
