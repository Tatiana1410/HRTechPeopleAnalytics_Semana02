# HRTech People Analytics - Equipo 8 - Semana 3

Sistema de analítica de datos y procesamiento de Big Data para la gestión del talento humano, análisis de desempeño y predicción de rotación laboral.

---

## 📁 Estructura del Proyecto

```text
HRTechPeopleAnalytics_Semana03/
│
├── datos/
│   ├── dataset_consolidado.csv         # Dataset raw consolidado
│   ├── dataset_consolidado.parquet     # Dataset en formato columnar comprimido
│   └── kpis_hrtech_departamentos.parquet # KPIs procesados por Spark
│
├── notebook/
│   ├── guia02.ipynb                    # Limpieza y preparación de datos
│   └── guia_03_bigdata_spark.ipynb     # Carga, conversión a Parquet y KPIs con PySpark
│
├── .gitignore
├── README.md                           # Documentación general del proyecto
└── requirements.txt                    # Dependencias del entorno Python