# Análisis y clasificación de reseñas de aplicaciones de citas

Trabajo final de **Desarrollo de Sistemas de Inteligencia Artificial — Grupo 2**.

Análisis de reseñas de **Tinder, Bumble y Hinge** para estudiar cómo se relacionan las valoraciones de los usuarios con los temas mencionados en sus comentarios. El proyecto incluye análisis exploratorio, cuatro hipótesis y la comparación de tres modelos de clasificación de texto.

📓 **[Abrir el notebook final](./TP_DSIA_Entrega_Final_Grupo2_v7.ipynb)**

## Objetivo

Comparar tipos y combinaciones de temas detectados, su asociación con las valoraciones y los cambios en las diferencias entre aplicaciones al igualar la composición temática. Complementariamente, evaluar cuánto permite el texto de una reseña anticipar una valoración alta o baja.

Los resultados pueden orientar la revisión de comentarios por equipos de producto y experiencia de usuario. La clasificación se utiliza como ejercicio predictivo: cuando ya se conocen las estrellas, no se necesita un modelo para agruparlas.

## Dataset

- **Fuente:** [Dating App Reviews: Tinder, Bumble & Hinge](https://www.kaggle.com/datasets/samartalwar/dating-app-reviews-tinder-bumble-and-hinge/data), publicado por `samartalwar` en Kaggle.
- **Origen declarado:** reseñas recopiladas de Google Play Store, agregadas y estandarizadas.
- **Archivo:** `dating_app_reviews_sentiment.csv`.
- **Dimensiones:** 10.709 reseñas y 9 columnas.
- **Período observado:** 18 de mayo al 13 de septiembre de 2026, con cobertura diferente según la aplicación.

| Variable            | Descripción                                         |
| ------------------- | --------------------------------------------------- |
| `app_name`          | Aplicación reseñada                                 |
| `review_text`       | Comentario libre                                    |
| `rating`            | Calificación en estrellas                           |
| `sentiment`         | Etiqueta positiva o negativa incluida en el archivo |
| `is_satisfied`      | Indicador de satisfacción incluido en el archivo    |
| `review_word_count` | Cantidad de palabras                                |
| `review_char_len`   | Cantidad de caracteres                              |
| `thumbs_up`         | Votos de utilidad registrados                       |
| `posted_at`         | Fecha y hora de publicación                         |

### Interpretación de las valoraciones

El conjunto permite estudiar valoraciones espontáneas de satisfacción, pero **no constituye una encuesta formal ni una muestra representativa** de todos los usuarios. Cada fila representa una reseña, no necesariamente una persona distinta.

La auditoría verifica que las etiquetas coinciden exactamente con esta agrupación de estrellas:

| Calificación  | `sentiment` | `is_satisfied` | Clase predictiva |
| ------------- | ----------- | -------------- | ---------------- |
| 1–2 estrellas | Negative    | 0              | Valoración baja  |
| 4–5 estrellas | Positive    | 1              | Valoración alta  |

No hay registros de 3 estrellas. La coincidencia de etiquetas no demuestra cómo se generaron originalmente ni acredita un análisis independiente del sentimiento del texto.

## Hipótesis y resultados descriptivos

| Hipótesis                                                                                                                                                | Resultado observado                                                                                                                                |
| -------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **H1.** Entre reseñas con un solo tema detectado, las técnicas presentan menor proporción de valoraciones altas que monetización y autenticidad/estafas. | Técnica: **3,50%** (371 casos); monetización: **7,48%** (2.073); autenticidad/estafas: **5,68%** (511). La tasa técnica es menor en las tres apps. |
| **H2.** Dos o más temas detectados se asocian con menor proporción de valoraciones altas que uno solo, incluso entre extensiones similares.              | Dos o más: **1,29%** (619 casos); uno: **6,67%** (2.955). La dirección se mantiene en los cuatro tramos de longitud.                               |
| **H3.** Las diferencias entre apps se reducen al comparar una composición temática común.                                                                | La brecha máximo–mínimo pasa de **21,38 a 18,93 puntos porcentuales**, una reducción de **2,46 pp** (aproximadamente 11,5%).                       |
| **H4.** Combinar monetización y autenticidad/estafas, sin técnica detectada, se asocia con menos valoraciones altas que cada tema por separado.          | Ambas: **1,15%** (520 casos); monetización sola: **7,48%**; autenticidad sola: **5,68%**.                                                          |

Son hipótesis **exploratorias reformuladas después de conocer los datos**. Reciben apoyo descriptivo en los contrastes principales; los resultados no establecen causas. H2 estudia acumulación general y H4 una combinación específica, de modo que no son evidencias independientes.

Los temas se identifican mediante diccionarios y expresiones regulares publicados en el notebook. **Un solo tema significa un solo tema detectado**, no exclusividad semántica. Una mención puede ser favorable, desfavorable o negada; detectar un tema no equivale a identificar una queja. Se incluyen ejemplos y desgloses por app y longitud.

Para H3 se emplea estandarización directa con pesos comunes, comparando tasas crudas y estandarizadas sobre las mismas filas. El análisis principal exige al menos 20 reseñas de cada app por perfil temático e incluye 10.610 reseñas; excluye 99. La sensibilidad con todos los perfiles y con el período común mantiene la reducción. La mayor parte de la brecha permanece, y el 11,5% no debe interpretarse como una proporción causalmente explicada.

Las valoraciones generales por aplicación se conservan como EDA complementario, en lugar de usarse como una hipótesis basada en cualquier diferencia mínima.

## Análisis y preparación de datos

El notebook desarrolla:

- Revisión de estructura, tipos, valores faltantes, duplicados y dominios.
- Conversión de `posted_at` a fecha y análisis de cobertura temporal.
- Análisis univariado de aplicaciones, estrellas, etiquetas, extensión y votos.
- Comparaciones bivariadas y multivariadas por aplicación, tema, período y longitud.
- Gráficos para responder las cuatro hipótesis.
- Auditoría de la redundancia entre estrellas y etiquetas.

Se conservaron las **10.709 filas**. No se encontraron nulos ni duplicados de fila completa. Los textos repetidos y los valores extremos no se eliminaron automáticamente, porque no existe evidencia suficiente para considerarlos errores. Los indicadores temáticos se incorporaron sin modificar el comentario original.

## Modelos y evaluación

### Tarea predictiva

Clasificación binaria de `valoracion_alta` a partir de **`review_text` exclusivamente**:

- **0:** rating de 1–2 estrellas.
- **1:** rating de 4–5 estrellas.

Se excluyen de las entradas `rating`, `sentiment`, `is_satisfied` y las variables auxiliares para evitar fuga de información y mantener una comparación centrada en el contenido textual.

### Protocolo

1. Agrupación de textos mediante una clave normalizada con Unicode NFKC, minúsculas y eliminación de espacios y puntuación.
2. Separación de desarrollo y prueba mediante `StratifiedGroupKFold`, con semilla 42, manteniendo cada grupo textual en una misma partición.
3. Validación cruzada de cuatro particiones por grupos sobre desarrollo.
4. Vectorización **TF-IDF** dentro de cada `Pipeline`: unigramas y bigramas, hasta 20.000 términos, `min_df=2` y frecuencia sublineal.
5. Selección por **F1 macro medio de validación** y evaluación posterior en la reserva de prueba.

La ejecución documentada utiliza **8.618 reseñas de desarrollo y 2.091 de prueba**, sin grupos textuales compartidos. La normalización controla coincidencias y variantes superficiales; no garantiza detectar paráfrasis ni identificar al mismo usuario.

### Comparación de resultados

| Modelo                  | F1 macro en validación | F1 macro en prueba | Accuracy en prueba | Balanced accuracy en prueba |
| ----------------------- | ----------------------:| ------------------:| ------------------:| ---------------------------:|
| Naive Bayes multinomial | 0,8110                 | 0,8295             | 0,8847             | 0,7905                      |
| Regresión logística     | 0,8951                 | 0,9041             | 0,9254             | 0,8996                      |
| **SVM lineal**          | **0,8991**             | **0,9043**         | **0,9254**         | **0,9007**                  |
| Línea base mayoritaria  | 0,4190                 | 0,4222             | 0,7308             | 0,5000                      |

**Modelo seleccionado: SVM lineal**, por su F1 macro medio de validación. Su ventaja sobre regresión logística es pequeña y no demuestra superioridad estadística.

En prueba, el recall de SVM es **0,9542 para valoraciones bajas** y **0,8472 para altas**. El notebook incluye precisión, recall y F1 por clase, matrices de confusión, métricas por aplicación, ejemplos de errores y términos con mayor peso en el clasificador.

## Cómo ejecutar

### 1. Preparar los archivos

Ubicar en la misma carpeta:

| Archivo                                 | Uso                                                            |
| --------------------------------------- | -------------------------------------------------------------- |
| `README.md`                             | Descripción del proyecto                                       |
| `TP_DSIA_Entrega_Final_Grupo2_v4.ipynb` | Notebook con el análisis y las salidas ejecutadas              |
| `dating_app_reviews_sentiment.csv`      | Datos de entrada; descargar desde Kaggle si no están incluidos |

El enlace al notebook de este README supone que ambos archivos están en la raíz del repositorio. El dataset conserva las condiciones de uso de su publicación original.

### 2. Crear un entorno virtual

```bash
python -m venv .venv
```

Activarlo según el sistema:

**Windows — PowerShell:**

```powershell
.\.venv\Scripts\Activate.ps1
```

**Linux o macOS:**

```bash
source .venv/bin/activate
```

### 3. Instalar dependencias

```bash
python -m pip install numpy==2.1.3 pandas==2.2.3 scikit-learn==1.6.1 matplotlib seaborn scipy ipython ipykernel jupyterlab
```

La ejecución de referencia se realizó con **Python 3.13.5**, pandas 2.2.3, NumPy 2.1.3 y scikit-learn 1.6.1. Las cifras de la tabla de modelos corresponden a esa ejecución de referencia, conservada al reformular las hipótesis. El análisis descriptivo actualizado puede recalcularse ejecutando todo el notebook. Las demás dependencias del comando no están fijadas; el entorno no es un bloqueo completo de versiones. Conservar las versiones principales ayuda a reproducir las particiones y métricas registradas.

### 4. Ejecutar el notebook

- **VS Code:** abrir el `.ipynb`, seleccionar el intérprete de `.venv` como kernel y ejecutar **Run All**.
- **JupyterLab:** ejecutar `jupyter lab`, abrir el notebook y ejecutar todas las celdas en orden.
- **Deepnote:** importar el notebook, subir el CSV y ejecutar todas las celdas con las dependencias disponibles.

El trabajo se ejecuta en CPU, sin GPU ni servicios pagos. Una vez instaladas las dependencias y descargados los datos, no requiere conexión a internet.

Las salidas ya están guardadas para su consulta. El hash SHA-256 de la copia analizada es:

```text
17b6b1e037b6f5a9ebd612889a73471959eab436bb8f0d4581549c663e3cd1db
```

## Limitaciones

- Las reseñas son autoseleccionadas y no representan a todos los usuarios.
- La cobertura temporal difiere entre aplicaciones y no hay valoraciones de 3 estrellas.
- Las etiquetas de satisfacción no constituyen una medición independiente del rating.
- El análisis temático usa vocabulario principalmente inglés y no mide cobertura por idioma.
- La reserva de prueba pertenece al mismo archivo, ya explorado durante el proyecto; no es una validación externa ni temporal independiente.
- Los textos pueden ser ambiguos o discrepar con las estrellas. No se verificó la experiencia relatada por cada usuario.

El resultado es un **prototipo académico**. Su aplicación a comentarios nuevos, sin estrellas, de otras aplicaciones o de otros períodos requiere validación adicional.

## Integrantes

- Aular Puerta Jose Miguel.
- Planes Maria Anabella.
- Romero Sofia Alejandra.
- Stancanelli Cristian Fabian.

## Referencias

- [Dataset original — samartalwar, Kaggle](https://www.kaggle.com/datasets/samartalwar/dating-app-reviews-tinder-bumble-and-hinge/data).
- [Notebook de referencia — Devra AI, Kaggle](https://www.kaggle.com/code/devraai/dating-app-reviews-analysis-and-prediction/notebook). Se consultó como antecedente; no se reproduce su uso de `rating` para predecir `is_satisfied`.
- [scikit-learn — StratifiedGroupKFold](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.StratifiedGroupKFold.html).
- [scikit-learn — Extracción de características textuales](https://scikit-learn.org/stable/modules/feature_extraction.html#text-feature-extraction).
- [scikit-learn — Métricas de evaluación](https://scikit-learn.org/stable/modules/model_evaluation.html#classification-metrics).
