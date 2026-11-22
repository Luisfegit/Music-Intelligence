# Music Intelligence

**Motor de inteligencia competitiva para pequeñas agencias musicales.**

Music Intelligence es un proyecto académico de Ciencia de Datos que
utiliza información de canciones y artistas de Spotify para construir
perfiles comparativos, identificar artistas similares y estimar la
popularidad observada de canciones mediante Machine Learning.

El propósito es apoyar a pequeñas agencias musicales en la
identificación de artistas y canciones que merecen una revisión
comercial más profunda. El sistema no pretende garantizar el éxito de
una canción ni predecir la rentabilidad de campañas.

## Problema y objetivo

Las pequeñas agencias musicales necesitan organizar información sobre
artistas y canciones para contextualizar su posicionamiento y priorizar
qué casos revisar con mayor detalle.

Music Intelligence propone un flujo analítico con dos componentes:

1.  **Diagnóstico competitivo de artistas:** resume la información
    disponible de cada artista y busca perfiles comparables.
2.  **Estimación de popularidad de canciones:** compara un modelo base
    con un modelo Random Forest y calcula la diferencia entre la
    popularidad observada y la estimada.

## Datos

El proyecto utiliza el archivo `spotify_data_clean_readable.csv`,
alojado en la raíz de este repositorio.

El conjunto de datos utilizado en el análisis contiene 8.582 registros
de canciones. En la preparación se excluyen registros sin nombre de
artista o sin valor válido de popularidad de canción.

### Criterio de cobertura de artistas

Para el diagnóstico competitivo se utiliza un umbral mínimo de **cinco
canciones por artista**. En el análisis realizado se identificaron 336
artistas que cumplen este criterio, correspondientes a 5.381 canciones.

El umbral se adopta como una decisión metodológica práctica: los
perfiles con muy pocas canciones ofrecen información limitada para
describir y comparar un catálogo. No constituye un estándar estadístico
universal.

El umbral se aplica al benchmarking de artistas, no al entrenamiento del
modelo de Machine Learning.

### Limitaciones de los datos

-   Una parte importante de los registros no contiene información de
    género musical.
-   Algunos atributos del artista pueden presentar valores distintos
    entre las canciones asociadas al mismo nombre; para construir
    perfiles se utilizan medianas.
-   La agrupación de artistas se realiza mediante nombres normalizados y
    no mediante un identificador único de artista. Por ello, pueden
    existir homónimos.
-   La muestra no debe interpretarse como una representación completa de
    Spotify o del mercado musical.

## Metodología

### 1. Perfiles y artistas comparables

Los perfiles se construyen mediante:

-   Mediana de popularidad del artista.
-   Mediana de seguidores.
-   Número de canciones distintas disponibles.
-   Unión de los géneros musicales registrados.

La búsqueda de artistas comparables considera la cercanía en
popularidad, seguidores y, cuando hay información disponible para ambos
perfiles, similitud de géneros. Los pesos utilizados son una regla de
diseño del prototipo y no parámetros aprendidos automáticamente.

### 2. Modelo de Machine Learning

La variable objetivo es `track_popularity`.

Se comparan dos enfoques:

-   **Baseline:** predice la mediana de popularidad del conjunto de
    entrenamiento.
-   **Random Forest Regressor:** estima la popularidad utilizando
    características numéricas y categóricas de la canción y del artista.

La evaluación utiliza:

-   **MAE:** error absoluto medio.
-   **RMSE:** raíz del error cuadrático medio.
-   **R²:** coeficiente de determinación.

La separación entre entrenamiento y prueba se realiza por grupos de
artista (`GroupShuffleSplit`) para evitar que canciones del mismo nombre
de artista aparezcan simultáneamente en ambos conjuntos. Se utiliza una
semilla fija para facilitar la reproducibilidad.

### 3. Desviación entre popularidad real y estimada

Para cada canción del conjunto de prueba se calcula:

`desviacion = popularidad_real - popularidad_estimada`

-   Una desviación positiva indica que la popularidad observada supera
    la estimación del modelo.
-   Una desviación negativa indica que la popularidad observada está por
    debajo de la estimación.

Estas desviaciones son señales exploratorias que pueden orientar una
revisión humana. No demuestran potencial de crecimiento futuro ni
rentabilidad comercial.

## Resultados del modelo

En una ejecución de referencia del notebook, con la partición definida
mediante `random_state=42`, se obtuvieron los siguientes resultados:

  Modelo                   MAE    RMSE       R²
  -------------------- ------- ------- --------
  Baseline (mediana)     20,13   25,98   -0,091
  Random Forest          15,37   20,90    0,294

En esa ejecución, Random Forest redujo el MAE aproximadamente un 23,7 %
frente al baseline.

Los resultados pueden cambiar si se modifican los datos, las variables,
el preprocesamiento o la partición. Por ello, las métricas definitivas
deben tomarse de la ejecución actual del notebook y del archivo
`comparacion_modelos.csv`.

## Estructura del repositorio

``` text
Music-Intelligence/
│
├── spotify_data_clean_readable.csv
├── Music_Intelligence_Modelo_Integrado.ipynb
├── README.md
│
└── resultados/
    ├── comparacion_modelos.csv
    ├── predicciones_random_forest_prueba.csv
    ├── artistas_validos_5_o_mas_canciones.csv
    └── perfiles_artistas.csv
```

La carpeta `resultados/` contiene archivos derivados del notebook. Si se
actualiza el dataset o el código, deben regenerarse estos archivos para
mantenerlos sincronizados.

## Cómo ejecutar el proyecto

### Requisitos

-   Python 3
-   Jupyter Notebook o JupyterLab
-   Las librerías: `pandas`, `numpy` y `scikit-learn`

Instalación de dependencias:

``` bash
pip install pandas numpy scikit-learn jupyter
```

### Ejecución

1.  Clonar o descargar este repositorio.
2.  Abrir `Music_Intelligence_Modelo_Integrado.ipynb` en Jupyter
    Notebook, JupyterLab o Visual Studio Code.
3.  Ejecutar las celdas en orden.
4.  El notebook lee el CSV directamente desde la URL pública `raw` de la
    rama `main`; no es necesario configurar una ruta local para el
    dataset.
5.  Al finalizar, el notebook genera los CSV de perfiles, métricas y
    predicciones en el directorio de trabajo.

Si el archivo CSV cambia de nombre, ubicación o rama, debe actualizarse
la variable `RUTA_DATOS` en la sección de configuración del notebook.

## Limitaciones e interpretación

-   La popularidad de Spotify es una métrica de plataforma y no equivale
    a ventas, ingresos, aceptación en vivo o éxito comercial.
-   El modelo estima popularidad observada a partir de los atributos
    disponibles; no predice popularidad futura.
-   La popularidad y los seguidores del artista se utilizan como
    variables de contexto, por lo que la estimación no mide únicamente
    características intrínsecas de la canción.
-   La disponibilidad incompleta de géneros y otros metadatos limita la
    calidad de algunas comparaciones.
-   La evaluación utiliza una partición de prueba por artista. Una sola
    partición no describe toda la variabilidad posible del desempeño del
    modelo.
-   Las canciones con desviaciones positivas o negativas deben revisarse
    en contexto antes de tomar decisiones promocionales.

## Tecnologías

-   Python
-   Pandas
-   NumPy
-   Scikit-learn
-   Jupyter Notebook
-   Git y GitHub

## Alcance académico

Este proyecto fue desarrollado como parte de una actividad introductoria
de Ciencia de Datos. Se priorizan la preparación reproducible de datos,
la comparación con un modelo base, la evaluación del modelo, la
interpretación de resultados y la comunicación de limitaciones.

------------------------------------------------------------------------

**Music Intelligence --- Data Science Project**
