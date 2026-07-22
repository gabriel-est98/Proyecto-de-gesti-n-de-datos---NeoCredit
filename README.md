# Calidad y Limpieza de Datos — Análisis NeoCredit 

Este repositorio contiene el desarrollo del Taller 3 sobre calidad y limpieza de datos, aplicado al caso de estudio **NeoCredit**. El objetivo principal de este proyecto es diagnosticar problemas de calidad, aplicar estrategias de limpieza justificadas, transformar variables financieras y validar de forma automatizada que el dataset cumple con las reglas de negocio, preparándolo para un futuro análisis de riesgo crediticio.

## Objetivo del Análisis
Aplicar técnicas modernas de detección y limpieza de datos utilizando el ecosistema de Python para asegurar la confiabilidad de los datos de solicitudes de crédito, antes de su uso en reportes de negocio o modelos de *machine learning* (Scoring).

## Dataset Utilizado
Se utilizó el dataset `solicitudes_credito_neocredit.csv`, el cual contiene **669 solicitudes de crédito** y **21 columnas** con información demográfica, laboral y financiera de los solicitantes.

## Principales Hallazgos y Decisiones de Limpieza

A continuación, se detalla el flujo de trabajo y las justificaciones de las decisiones tomadas durante el análisis, basándonos en el perfilado de datos:

### 1. Tratamiento de Datos Faltantes y Duplicados
* **Diagnóstico:** Se descartó la eliminación masiva de filas o columnas con nulos, ya que se perdería un porcentaje inaceptable de solicitudes válidas o variables críticas para el negocio.
* **Corrupción de Datos:** Se identificaron y eliminaron **6 filas totalmente corruptas** (con nulos simultáneos en casi todas las columnas numéricas clave, como `monto_solicitado` y `edad`).
* **Duplicados:** Se detectaron y eliminaron solicitudes duplicadas basadas en el identificador único `id_solicitud`.
* **Imputación:** Se utilizó `SimpleImputer` de `scikit-learn` para rellenar los valores faltantes de forma reproducible. Se usó la **mediana** para variables numéricas (robusta ante atípicos) y la **moda** para variables categóricas.

### 2. Corrección de Inconsistencias de Tipo y Reglas de Negocio
* **Símbolos y Texto en Números:** La variable `ingreso_mensual` presentaba símbolos de moneda (`$`) y comas, lo que impedía su tratamiento matemático. Se limpiaron estos caracteres y se forzó su conversión a formato numérico, enviando los valores negativos a `NaN` por violar la regla de negocio.
* **Normalización Categórica:** Columnas como `genero`, `ciudad`, `tipo_empleo`, `canal_solicitud` y `resultado_solicitud` presentaban inconsistencias de capitalización, espacios sobrantes y sinónimos (ej. *F*, *f*, *Femenino*). Se estandarizaron utilizando diccionarios de mapeo y funciones de *string* de pandas.
* **Gestión de Fechas (Formatos Mixtos):** La columna `fecha_solicitud` presentaba un desafío complejo: **cuatro formatos distintos mezclados** (ISO `aaaa-mm-dd`, Latino `dd/mm/aaaa`, Estadounidense `mm-dd-aaaa` y Texto `"18 de November de 2025"`). Se desarrolló una función personalizada basada en expresiones regulares (`regex`) para parsear correctamente cada formato sin pérdida de datos por ambigüedad de días/meses.
* **Alertas de Riesgo:** Se extrajo la etiqueta `(VPN)` de la columna `ip_pais` hacia una nueva variable indicadora de riesgo (`ip_vpn_flag`).
* **Valores Fuera de Rango:** Se detectaron edades inválidas (ej. -5 o 210 años), antigüedades negativas y *scores* de buró negativos. Estos fueron tratados como valores faltantes previo a la imputación.

### 3. Escalado y Normalización
Se aplicaron transformaciones de `scikit-learn` para preparar las variables financieras de cara a futuros modelos sensibles a la escala:
* **Escalado:** Se optó por utilizar `RobustScaler` para la variable `ingreso_mensual`, ya que es menos sensible a los valores atípicos reales (ingresos muy altos) en comparación con `MinMaxScaler`. También se utilizó `MaxAbsScaler` en el `monto_solicitado`.
* **Normalización:** Se aplicó la transformación Box-Cox (mediante `PowerTransformer`) al `monto_solicitado` para estabilizar su varianza, y `StandardScaler` al `score_buro_externo` para estandarizar su distribución.

### 4. Validación Automatizada (Contratos de Datos)
Se definió un esquema de validación declarativa usando la librería `pandera` para asegurar la calidad final del dataset. Las reglas exigieron:
* Unicidad del `id_solicitud`.
* Rangos estrictos: `edad` (18-100), `score_buro_externo` (0-999).
* Valores estrictamente positivos para el `ingreso_mensual` y el `monto_solicitado`.
* Categorías cerradas para `plazo_meses` y `resultado_solicitud` ("Aprobado", "Rechazado", "Pendiente").

## Tecnologías y Librerías Utilizadas
* **Lenguaje:** Python 3 (entorno Google Colab)
* **Manipulación de Datos:** `pandas`, `numpy`
* **Diagnóstico Visual:** `ydata-profiling`, `missingno`
* **Transformación e Imputación:** `scikit-learn` (`SimpleImputer`, `RobustScaler`, `PowerTransformer`, etc.)
* **Validación de Calidad:** `pandera`
