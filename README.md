EA3 práctico — Aprendizaje no supervisado en scoring crediticio

Contexto y objetivo
- Se implementa un método no supervisado (K-Means) sobre el conjunto de entrenamiento del proyecto de scoring crediticio.
- Propósito: identificar segmentos homogéneos de clientes y relacionarlos con tasas de mora y, opcionalmente, con probabilidades de incumplimiento estimadas por el modelo supervisado.
- Se sigue la lógica CRISP-DM, evitando data leakage: solo se usa el dataset de entrenamiento.

Técnica y justificación
- K-Means: agrupa observaciones en `k` clusters minimizando la varianza intra-cluster.
- Ventajas: sencillo, escalable, útil para segmentación inicial; produce clusters comparables con métricas estándar como `silhouette`.
- Limitaciones: requiere elección de `k`, asume clusters convexos; se mitiga con evaluación y visualización PCA.

Datos y variables
- Entrada: CSV del conjunto de entrenamiento (`--train-csv`).
- Columnas de trabajo: por defecto se seleccionan numéricas; opcional `--feature-columns` para especificarlas.
- Columna objetivo opcional (`--target-column`): usada solo para calcular tasa de mora por cluster; no participa en el clustering.
- Columna identificadora (`--id-column`) opcional para vincular predicciones supervisadas.
- Predicciones supervisadas opcionales (`--predictions-path`) con columna `pred_proba` para promedios por cluster.

Evaluación dentro del proyecto
- Métrica interna: `silhouette` del clustering.
- Análisis de utilidad: tasa de mora por cluster, tamaño, y si se proveen predicciones, promedio de PD por cluster.
- Visualizaciones: dispersión PCA 2D coloreada por cluster y por mora.
- Potencial integración: usar clusters como features del modelo supervisado, o como segmentos para calibración/monitoreo de riesgo.

Instrucciones de ejecución (Notebook)
1) Instalar dependencias mínimas:
   - `pip install pandas numpy scikit-learn matplotlib seaborn jupyter`
2) Abrir Jupyter en la raíz del repositorio:
   - `python -m jupyter notebook`
3) Abrir `unsupervised/kmeans_clustering.ipynb` y ajustar la celda de configuración si corresponde:
   - `TRAIN_CSV = 'data/train.csv'` (el notebook intenta automáticamente `../data/train.csv` si se ejecuta desde `unsupervised/`)
   - `TARGET_COLUMN = 'mora'` y `ID_COLUMN = 'customer_id'`
   - `OUT_DIR = 'outputs/unsupervised/kmeans_nb'`
4) Ejecutar las celdas en orden. El notebook buscará el mejor `k` por `silhouette` y generará:
   - `outputs/unsupervised/kmeans_nb/config.json`
   - `outputs/unsupervised/kmeans_nb/metrics.json` (incluye `silhouette`)
   - `outputs/unsupervised/kmeans_nb/k_search.csv` y `silhouette_vs_k.png`
   - `outputs/unsupervised/kmeans_nb/cluster_assignments.csv`
   - `outputs/unsupervised/kmeans_nb/cluster_summary.csv`
   - `outputs/unsupervised/kmeans_nb/clusters_pca.png`, `default_pca.png`

Interpretación de resultados
- Clusters con mayores tasas de mora o mayores PD promedio pueden señalar subpoblaciones de mayor riesgo.
- Diferencias claras entre clusters sugieren que segmentar puede mejorar calibración, pricing o límites de crédito.
- `silhouette` alto indica separación razonable; valores bajos motivan ajustar `k` o features.

Discusión sobre incorporación al proyecto
- Integración directa: añadir el índice de cluster como feature al modelo supervisado y validar mejora.
- Monitoreo: usar el resumen por cluster para seguimiento de estabilidad y drift poblacional.
- Consideraciones: elegir `k` con criterio (por ejemplo, validando con estabilidad temporal), revisar impacto en sesgos y fairness.

Notas de uso en GitHub
- Usa rutas relativas (`data/train.csv`, `outputs/...`) para portabilidad.
- Abre Jupyter en la raíz del repo; si ejecutas desde `unsupervised/`, el notebook aplica *fallback* a `../data/train.csv`.
- Incluye `data/train.csv` o documenta cómo generarlo/obtenerlo para reproducibilidad.

Resultados (visualizaciones y archivos)
- Curva de evaluación: `outputs/unsupervised/kmeans_nb/silhouette_vs_k.png`
- Clusters en PCA 2D: `outputs/unsupervised/kmeans_nb/clusters_pca.png`
- Mora sobre PCA: `outputs/unsupervised/kmeans_nb/default_pca.png`
- Búsqueda de k: `outputs/unsupervised/kmeans_nb/k_search.csv`
- Resumen por cluster: `outputs/unsupervised/kmeans_nb/cluster_summary.csv`
- Asignaciones: `outputs/unsupervised/kmeans_nb/cluster_assignments.csv`

Vista rápida en GitHub (si ya ejecutaste el notebook y generaste salidas):
- ![Silhouette vs k](outputs/unsupervised/kmeans_nb/silhouette_vs_k.png)
- ![Clusters PCA](outputs/unsupervised/kmeans_nb/clusters_pca.png)
- ![Mora en PCA](outputs/unsupervised/kmeans_nb/default_pca.png)

Notas de buenas prácticas
- Evitar leakage: usar exclusivamente el CSV de entrenamiento para clustering y para las predicciones opcionales.
- Documentar qué columnas se utilizaron y registrar la configuración en `config.json`.
- Versionar salidas y reproducibilidad fijando `--random-state`.
