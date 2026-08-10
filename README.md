# HRTechPeopleAnalytics_Semana02


## Descripción del proyecto

Este proyecto corresponde al proceso de limpieza de datos e ingeniería de características aplicado a un dataset consolidado de Recursos Humanos.

El objetivo principal es preparar un conjunto de datos para su posterior análisis, realizando diagnóstico de calidad, normalización, corrección de valores inválidos, tratamiento de duplicados, conversión de fechas y creación de nuevas características relacionadas con el comportamiento y estado de los colaboradores.

---

## Estructura del proyecto

```text
HRTechPeopleAnalytics_Semana02/
│
├── datos/
│   ├── dataset_consolidado.csv
│   └── dataset_limpio_guia02.csv
│
├── notebook/
│   └── guia02.ipynb
│
├── .venv/
│
└── README.md
```

---

## Configuración del entorno virtual

### 1. Crear el entorno virtual

Desde la terminal de Visual Studio Code:

```powershell
python -m venv .venv
```

---

### 2. Activar el entorno virtual

En PowerShell:

```powershell
.\.venv\Scripts\Activate.ps1
```

Cuando el entorno esté activo, debería aparecer algo similar a:

```text
(.venv)
```

al inicio de la línea de comandos.

---

## Instalación de dependencias

### 3. Actualizar pip

```powershell
python -m pip install --upgrade pip
```

---

### 4. Instalar Jupyter e IPython Kernel

```powershell
python -m pip install jupyter ipykernel
```

---

### 5. Instalar librerías utilizadas

```powershell
python -m pip install pandas matplotlib numpy
```

Las principales librerías utilizadas son:

* `pandas`: manipulación y análisis de datos.
* `numpy`: operaciones numéricas y tratamiento de valores nulos.
* `matplotlib`: generación de visualizaciones.
* `jupyter`: ejecución de notebooks.
* `ipykernel`: integración del entorno virtual con Jupyter.

---

# Proceso desarrollado

## 1. Definición de rutas

Se definió una carpeta principal denominada `datos`, donde se almacena tanto el dataset original como el dataset limpio generado al finalizar el proceso.

```python
from pathlib import Path
import pandas as pd
import numpy as np

BASE_DIR = Path("../")
DATOS = BASE_DIR / "datos"

DATOS.mkdir(parents=True, exist_ok=True)
```

---

## 2. Carga del dataset

El dataset consolidado se carga utilizando `pandas`.

```python
ruta_entrada = DATOS / "dataset_consolidado.csv"

df = pd.read_csv(ruta_entrada)

df.head()
```

---

## 3. Diagnóstico inicial de calidad

Se realizaron las siguientes verificaciones:

* cantidad de filas y columnas;
* tipos de datos;
* valores nulos;
* porcentaje de valores nulos;
* registros duplicados.

Ejemplo:

```python
print("Filas y columnas:", df.shape)

df.dtypes

df.isnull().sum()

duplicados = df.duplicated().sum()

print("Registros duplicados:", duplicados)
```

---

## 4. Normalización de nombres de columnas

Los nombres de las columnas se normalizaron para evitar problemas posteriores.

```python
df.columns = (
    df.columns
    .str.strip()
    .str.lower()
    .str.replace(" ", "_")
)
```

---

## 5. Normalización de campos de texto

Se eliminaron espacios innecesarios al inicio y al final de las columnas de texto.

```python
columnas_texto = df.select_dtypes(include="object").columns

for columna in columnas_texto:
    df[columna] = df[columna].astype("string").str.strip()
```

---

## 6. Conversión de fechas

Las columnas que representan fechas fueron transformadas al tipo `datetime`.

```python
df["fecha_ingreso"] = pd.to_datetime(
    df["fecha_ingreso"],
    errors="coerce"
)

df["fecha_salida"] = pd.to_datetime(
    df["fecha_salida"],
    errors="coerce"
)

df["fecha_evaluacion"] = pd.to_datetime(
    df["fecha_evaluacion"],
    errors="coerce"
)

df["fecha_encuesta"] = pd.to_datetime(
    df["fecha_encuesta"],
    errors="coerce"
)
```

El parámetro `errors="coerce"` permite transformar fechas inválidas en valores `NaT`.

---

## 7. Corrección de valores inválidos

Se verificaron y corrigieron valores que no cumplían con reglas lógicas del dataset.

Entre las validaciones realizadas se encuentran:

* edades fuera de rango;
* horas de capacitación negativas;
* ausencias negativas;
* puntajes de desempeño fuera del rango esperado;
* respuestas de encuestas fuera de la escala de 1 a 5;
* fechas de salida anteriores a la fecha de ingreso.

Los valores inválidos fueron convertidos a valores nulos para poder ser tratados posteriormente.

---

## 8. Tratamiento de valores nulos

Los valores nulos fueron analizados según el significado de cada variable.

En el caso de `fecha_salida`, los valores nulos fueron conservados porque representan colaboradores que continúan activos en la organización.

Por esta razón, un valor nulo en esta columna no necesariamente representa un problema de calidad.

---

## 9. Eliminación de registros duplicados

Los registros completamente duplicados fueron eliminados.

```python
df = df.drop_duplicates()
```

Posteriormente se verificó nuevamente la cantidad de duplicados.

```python
print("Duplicados restantes:", df.duplicated().sum())
```

---

# Ingeniería de características

A partir de las variables originales se crearon cinco nuevas características.

## 10. Antigüedad

La variable `antiguedad` representa la cantidad de años completos que el colaborador lleva trabajando en la organización.

Se calcula a partir de `fecha_ingreso`.

```python
fecha_referencia = pd.Timestamp("2026-08-09")

df["antiguedad"] = (
    fecha_referencia.year - df["fecha_ingreso"].dt.year
)

df["antiguedad"] = df["antiguedad"] - (
    (fecha_referencia.month < df["fecha_ingreso"].dt.month) |
    (
        (fecha_referencia.month == df["fecha_ingreso"].dt.month) &
        (fecha_referencia.day < df["fecha_ingreso"].dt.day)
    )
)
```

---

## 11. Capacitaciones recibidas

La cantidad estimada de capacitaciones se calcula utilizando las horas de capacitación.

Se considera una capacitación por cada 8 horas registradas.

```python
df["capacitaciones_recibidas"] = (
    df["horas_capacitacion"] / 8
).round().astype("Int64")
```

---

## 12. Nivel de desempeño calculado

Se creó una nueva clasificación utilizando el puntaje de desempeño.

```python
def clasificar_desempeno(puntaje):
    if puntaje < 60:
        return "Bajo"
    elif puntaje < 80:
        return "Medio"
    else:
        return "Alto"


df["nivel_desempeno_calculado"] = (
    df["puntaje_desempeno"].apply(clasificar_desempeno)
)
```

---

## 13. Riesgo de rotación

El riesgo de rotación se determina utilizando información relacionada con:

* satisfacción laboral;
* intención de permanecer;
* días de ausencia.

La clasificación final puede ser:

```text
Bajo
Medio
Alto
```

Esta variable permite identificar colaboradores que podrían presentar mayor probabilidad de abandonar la organización.

---

## 14. Estado del colaborador

La variable `estado_colaborador` permite identificar si una persona continúa activa o ya salió de la organización.

La decisión se basa principalmente en:

* `estado_laboral`;
* `fecha_salida`.

Los posibles resultados son:

```text
Activo
Inactivo
```

---

# Reporte de calidad

Después del proceso de limpieza se generó un reporte para cada columna.

```python
reporte_calidad = pd.DataFrame({
    "columna": df.columns,
    "tipo_dato": df.dtypes.astype(str).values,
    "valores_nulos": df.isnull().sum().values,
    "porcentaje_nulos": (df.isnull().mean() * 100).round(2).values,
    "valores_unicos": df.nunique().values
})

reporte_calidad
```

El reporte permite analizar:

* tipo de dato;
* cantidad de valores nulos;
* porcentaje de valores nulos;
* cantidad de valores únicos.

---

# Decisiones tomadas

Durante el proceso de limpieza se tomaron las siguientes decisiones:

1. El dataset original se conservó sin modificaciones.
2. Las transformaciones se realizaron sobre el DataFrame `df`.
3. Los nombres de las columnas fueron normalizados.
4. Los campos de texto fueron limpiados de espacios innecesarios.
5. Las columnas correspondientes a fechas fueron convertidas a `datetime`.
6. Los valores inválidos fueron transformados en nulos antes de su tratamiento.
7. Los registros duplicados fueron eliminados.
8. Los valores nulos de `fecha_salida` fueron conservados porque representan colaboradores que continúan activos.
9. La antigüedad fue calculada en años completos.
10. Las capacitaciones recibidas fueron estimadas a partir de las horas de capacitación.
11. Se creó una clasificación de desempeño basada en el puntaje.
12. Se generó un indicador de riesgo de rotación utilizando variables relacionadas con satisfacción, permanencia y ausencias.
13. Se creó una variable para representar el estado actual del colaborador.

---

# Dataset resultante

Finalmente, el dataset limpio se guarda en un nuevo archivo para conservar el dataset original.

```python
ruta_salida = DATOS / "dataset_limpio_guia02.csv"

df.to_csv(
    ruta_salida,
    index=False,
    encoding="utf-8"
)

print("Dataset limpio guardado en:")
print(ruta_salida)
```

El archivo generado se encuentra en:

```text
datos/dataset_limpio_guia02.csv
```

---

## Resultado final

El proceso desarrollado permite transformar un dataset consolidado de Recursos Humanos en un conjunto de datos limpio, estructurado y enriquecido con nuevas características.

Las nuevas variables generadas son:

```text
antiguedad
capacitaciones_recibidas
nivel_desempeno_calculado
riesgo_rotacion
estado_colaborador
```

Estas características pueden ser utilizadas posteriormente para análisis exploratorio de datos, visualización, People Analytics o desarrollo de modelos de Machine Learning.
