# Fundamentos de Data Science (UPC 2026-1)

<p align="center">
  <img src="https://upload.wikimedia.org/wikipedia/commons/f/fc/UPC_logo_transparente.png" alt="Logo UPC" width="220"/>
</p>

## TB2 - Trabajo Final
### **Tema: Creación de conocimiento aplicando la metodología CRISP-DM**
**Caso de Estudio:** Análisis y Modelado de Tendencias de YouTube en la India

---

## 📌 1. Objetivo del Proyecto

El proyecto se desarrolla aplicando la metodología estructurada **CRISP-DM** (Cross-Industry Standard Process for Data Mining) para transformar datos crudos en conocimiento útil para la toma de decisiones estratégicas de una empresa de marketing digital. Los objetivos se dividen en dos niveles:

### Objetivo de Negocio
* **Análisis Descriptivo:** Analizar el conjunto de datos de videos en tendencia (Trending) de YouTube en la India para identificar patrones de consumo, evolución temporal del volumen de videos, preferencias de la audiencia (ratios de aceptación y pasividad) y distribución geográfica (a nivel de estados de la India).
* **Optimización Operativa:** Identificar las franjas horarias y días de publicación más eficientes para maximizar el alcance orgánico, así como asociarse con canales estratégicos según las metas presupuestarias de las campañas.

### Objetivo de Data Science (Modelado Predictivo)
* **Predicción de Engagement:** Desarrollar y evaluar un modelo de Machine Learning (**Regresión Lineal Múltiple**) para determinar la viabilidad de predecir el éxito cuantitativo de un video (medido en la cantidad de **Likes**) en función de sus métricas de interacción iniciales (`views`, `dislikes`, `comment_count`) y la temática del video (`nombre_categoria` codificada mediante variables *dummy*).

---

## 👥 2. Integrantes del Grupo y Roles

La asignación de roles y responsabilidades se definió basándose en las competencias técnicas de cada integrante para cubrir de forma integral las fases de la metodología **CRISP-DM**:

| Integrante | Rol Asignado | Responsabilidad Principal (CRISP-DM) |
| :--- | :--- | :--- |
| **Victor Piero Arapa Titi** | **Data Analyst** | Fase de Comprensión de Datos y Exploración (EDA), visualización y generación de *insights* de negocio. |
| **Rai Jeferson Manrique Anaya** | **Data Engineer** | Fase de Preparación de Datos. Procesamiento, limpieza técnica, corrección de formatos y estructuración del dataset. |
| **Alvaro Alberto Neyra Salazar** | **Data Scientist** | Fase de Modelado y Evaluación. Modelado predictivo, normalización/escalamiento de variables, métricas de regresión y validación. |
| **Josue Froilan Laurente Castrejon** | **Business Project Sponsor** | Fase de Comprensión del Negocio y Evaluación. Alineación de objetivos de negocio, formulación de preguntas clave y validación final de resultados con el cliente. |

---

## 📊 3. Descripción del Conjunto de Datos

El análisis se centra en el conjunto de datos de la región de la India, conformado por el archivo de registros de videos `INvideos.csv` y su respectivo archivo JSON de categorías `IN_category_id.json`. 

El conjunto de datos contiene variables cualitativas y cuantitativas sobre las interacciones y el origen geográfico.

### Estructura de Variables del Dataset Original
* **`video_id`** (Categorical): Identificador único de cada video en YouTube.
* **`trending_date`** (Date): Fecha en la que el video se convirtió en tendencia.
* **`title`** (Categorical): Título de publicación del video.
* **`channel_title`** (Categorical): Nombre del canal de YouTube creador.
* **`category_id`** (Integer): Identificador numérico de la categoría del video.
* **`publish_time`** (Date): Fecha y hora exacta de publicación del video.
* **`tags`** (Categorical): Etiquetas o palabras clave asignadas por el creador.
* **`views`** (Integer): Número total de visualizaciones acumuladas.
* **`likes`** (Integer): Reacciones positivas recibidas.
* **`dislikes`** (Integer): Reacciones negativas recibidas.
* **`comment_count`** (Integer): Número total de comentarios.
* **`thumbnail_link`** (Categorical): Enlace directo a la miniatura del video.
* **`comments_disabled`** (Boolean): Indicador de si la sección de comentarios fue desactivada.
* **`ratings_disabled`** (Boolean): Indicador de si las calificaciones de me gusta/no me gusta están ocultas.
* **`video_error_or_removed`** (Boolean): Indica si el video sufrió algún error de carga o fue retirado.
* **`description`** (Categorical): Texto detallado de la descripción del video.
* **`state`** (Categorical): Estado de la India asignado al registro de tendencia.
* **`lat`** (Float): Latitud geográfica del centro del Estado asignado.
* **`lon`** (Float): Longitud geográfica del centro del Estado asignado.
* **`geometry`** (Geometry): Coordenadas espaciales del límite del Estado.

### Variables Derivadas Creadas para el Análisis
* **`nombre_categoria`** (Categorical): Nombre descriptivo de la categoría del video (mapeado desde el JSON, corrigiendo la omisión de la categoría `29` como *"Nonprofits & Activism"*).
* **`dias_hasta_tendencia`** (Integer): Días transcurridos (en días) entre la fecha de publicación y el ingreso a la lista de tendencias.
* **`ratio_likes_dislikes`** (Float): Métrica de aceptación relativa (Likes / Dislikes).
* **`ratio_vistas_comentarios`** (Float): Métrica de pasividad de la audiencia (Views / Comments).

### 🛠️ Resumen del Proceso de Calidad y Limpieza de Datos
1. **Tratamiento de Desplazamiento por Saltos de Línea:** Se eliminaron **1,181 filas inválidas** en las cuales los saltos de línea sin comillas en el campo de texto libre `description` desplazaban los textos hacia columnas métricas, dejando valores nulos (`NaN`) en variables cuantitativas.
2. **Corrección de IDs de Excel:** Se identificaron y corrigieron **511 registros** con IDs corruptos (`#NAME?` y `#VALUE!`) causados por la autointerpretación de fórmulas en Excel. Se recuperaron IDs reales a partir de cruces históricos o se generó un hash único basado en `title` y `channel_title`.
3. **De-duplicación del Dataset Descriptivo:** Para evitar redundancias temporales (un video que es tendencia varios días), se filtró el dataset manteniendo únicamente el registro con mayor volumen de visitas (`views`) de cada video por día.
4. **Tratamiento de Outliers (Winsorización al $P_{99}$):** Con el objetivo de estabilizar el modelo de Regresión Lineal frente a la viralidad extrema de videos extraordinarios (outliers con millones de visualizaciones), se aplicó Winsorización al percentil 99 en las variables cuantitativas del dataset de modelado:
   * `views` acotado a un máximo de **12.35M**.
   * `likes` acotado a un máximo de **406.04k**.
   * `dislikes` acotado a un máximo de **19.47k**.
   * `comment_count` acotado a un máximo de **39.58k**.

---

## 📈 4. Conclusiones del Proyecto

A continuación se presentan las conclusiones del modelo de regresión lineal múltiple organizadas bajo la estructura analítica de validación cuantitativa y su implicación para el negocio:

### ⚙️ Conclusiones Técnicas y Validación de Métricas

* **MAE = 8,431.74**
  El Error Absoluto Medio (MAE) indica que, en promedio, las predicciones del modelo se desvían en aproximadamente 8,432 unidades de reacción (likes) respecto al valor real de los videos en tendencias. En el contexto de YouTube en la India, donde los videos virales analizados promedian decenas de miles de interacciones y alcanzan un tope de escala de 406,040 likes en el dataset de modelado, una desviación absoluta de solo 8.4k es notablemente baja en términos relativos. Esto valida la alta precisión y exactitud del modelo para realizar predicciones realistas sobre el comportamiento del engagement promedio.

* **RMSE = 19,587.40**
  La Raíz del Error Cuadrático Medio (RMSE) es una métrica de dispersión que mide el error típico del modelo, dando una penalización mayor a los errores de gran magnitud debido a que eleva los residuos al cuadrado antes de promediarlos. Al obtener un RMSE de aproximadamente 19,587 unidades de interacción (likes), que se mantiene en un rango controlado y bajo en comparación con los valores máximos de la muestra, se valida la consistencia y estabilidad de la regresión lineal. Esto confirma que el tratamiento de outliers contuvo eficazmente los errores severos y que las predicciones no sufren de desviaciones catastróficas.

* **MSE = 383,666,361.97**
  El Error Cuadrático Medio (MSE) representa el promedio de los errores al cuadrado expresado en unidades de likes al cuadrado (likes²). Aunque esta métrica no posee una interpretación directa y sencilla para el lenguaje común de negocio debido a sus unidades cuadráticas, sirve para validar el comportamiento del optimizador matemático de mínimos cuadrados, el cual minimiza esta pérdida durante el entrenamiento del modelo. Su valor estable confirma que el algoritmo convergió exitosamente hacia una solución matemática robusta y libre de problemas severos de multicolinealidad.

* **Análisis General**
  La validación conjunta de estas métricas demuestra de manera contundente que el modelo de regresión lineal múltiple posee una alta fiabilidad predictiva y matemática. Al mismo tiempo, la estrecha relación entre los bajos márgenes de error promedio (MAE) y típico (RMSE) en comparación con el rango global del dataset valida que la regresión realiza inferencias precisas y seguras tanto en videos de consumo moderado como en videos altamente virales. En consecuencia, el modelo está formalmente validado y listo para ser utilizado como una herramienta estratégica de negocio que permita proyectar el alcance de inversiones publicitarias y evaluar el rendimiento de contratos de patrocinio.

### 🏢 Conclusiones de Negocio
* **Predictibilidad del Éxito:** La validez matemática del modelo dota al cliente de una herramienta científica y estructurada para planificar el retorno de inversión (ROI) en campañas de marketing y estimar con seguridad el alcance esperado en contratos de patrocinio.
* **Centralización del Mercado:** A pesar del dominio de las grandes cadenas televisivas e institucionales en las tendencias, se valida que es posible identificar nichos de alta aceptación (como *Education* y *Pets & Animals*) con audiencias altamente comprometidas a un menor costo publicitario.
* **Optimización Operativa:** La regularidad de las tendencias sugiere la planificación estratégica de subidas de videos en horarios específicos para maximizar la difusión algorítmica inicial.

### ⚠️ Limitaciones y Trabajo Futuro
* **Representatividad Geográfica:** Las coordenadas y estados muestran dispersión artificial y homogénea debido a una asignación aleatoria posterior sobre la data original, por lo que no reflejan patrones de consumo territorial reales.
* **Modelos no Lineales:** Como trabajo futuro, se recomienda explorar algoritmos no lineales para capturar posibles relaciones dinámicas complejas que optimicen el ajuste en los límites del conjunto de datos.

---

## 📄 5. Licencia de Uso

Este proyecto se distribuye bajo la **Licencia MIT**. Esto permite el uso libre, la modificación, copia y distribución del código para fines académicos, de investigación y comerciales, siempre y cuando se incluya la atribución correspondiente a los autores originales de la Universidad Peruana de Ciencias Aplicadas (UPC).
