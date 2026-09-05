# Cheat sheet de pandas para análisis de datos

Guía de referencia en español para limpiar, diagnosticar y analizar datos con **pandas**, **matplotlib** y **seaborn**. Está escrita para consultarse en medio del trabajo, no para leerse de corrido.

> **Regla de oro: no se memoriza, se consulta.** Busca el patrón, copia el esqueleto, sustituye tus nombres de columna.

## Cómo está organizada

Cada tema sigue siempre la misma estructura:

- **Cuándo usarlo** — qué pregunta responde esa herramienta.
- **Código de ejemplo** — con nombres de columna concretos, para que se entienda.
- **🔁 Sintaxis genérica** — la misma línea con `nombre_columna`, lista para copiar y adaptar.
- **⚠️ Advertencias** — los errores que de verdad aparecen: el paréntesis de más, el `if` sobre una Serie, la conclusión de negocio construida sobre valores imputados.
- **Validación** — cómo comprobar que el paso hizo lo que creías.

Las secciones están numeradas del **1 al 45** y esa numeración es estable: las referencias cruzadas del tipo *"ver sección 22"* apuntan siempre al mismo tema, aunque cambie el archivo donde vive.

## Módulos

| Módulo | Contenido | Secciones |
|---|---|---|
| **[Carga e inspección inicial](docs/01-carga-e-inspeccion.md)** | Leer archivos, primer vistazo al DataFrame, seleccionar y reordenar columnas. | 1–5 |
| **[Valores nulos y duplicados](docs/02-nulos-y-duplicados.md)** | Diagnosticar huecos, decidir entre eliminar e imputar, quitar duplicados y auditar el resultado. | 6–15 |
| **[Tipos, texto y fechas](docs/03-tipos-texto-y-fechas.md)** | Normalizar strings, convertir texto a número y a `datetime` sin romper filas. | 16–20 |
| **[Filtrado y estadística descriptiva](docs/04-filtrado-y-estadistica.md)** | Condiciones combinadas, `groupby`, agregaciones y ordenamientos. | 21–24 |
| **[Combinar datasets y exportar](docs/05-merge-y-exportacion.md)** | Preparar claves, validar un `merge`, ordenar la tabla final y guardarla. | 25–29 |
| **[Diagnóstico de calidad e imputación](docs/06-calidad-de-datos.md)** | Roles vs. tipos, cardinalidad, sentinels, MCAR/MAR/MNAR y evaluación pre–post. | 30, 31 |
| **[Python aplicado a la limpieza](docs/07-python-para-limpieza.md)** | Funciones, bucles, `if`/`elif` y el pipeline completo `clean_data(df)`. | 32–35, 43 |
| **[Distribuciones e histogramas](docs/08-distribuciones.md)** | Forma, sesgo, `bins`, medidas categóricas y cuándo una forma NO es un hallazgo. | 36–38, 40, 45 |
| **[Outliers: detección y tratamiento](docs/09-outliers.md)** | IQR, Z-score, cuándo usar cada uno, y KEEP / DROP / CAP. | 39, 41, 42 |
| **[Feature engineering](docs/10-feature-engineering.md)** | `np.where()`, `.apply()`, `&` vs `and` y el error de segmentar al nivel equivocado. | 44 |

## El dataset de ejemplo

Los ejemplos concretos usan un dataset ficticio de ventas minoristas (~5,000 filas) con estas columnas:

| Columna | Rol | Notas |
|---|---|---|
| `order_id`, `customer_id` | ID | Numéricos, pero no se promedian |
| `order_date` | Fecha | Requiere conversión a `datetime` |
| `customer_age` | Numérica | Trae el sentinel `-999`; distribución uniforme (dato sintético) |
| `price`, `order_value` | Numérica | Muy sesgadas — el caso típico de outliers |
| `quantity` | Numérica discreta | Trae valores `<= 0` inválidos |
| `product_category` | Categórica | Trae el sentinel de texto `"?"` |
| `city`, `state`, `payment_method` | Categórica | Baja cardinalidad |

No hace falta tener el archivo: **el valor está en los patrones, no en los datos.** Sustituye los nombres por los de tu propio dataset y todo funciona igual.

## Requisitos

```bash
pip install pandas matplotlib seaborn numpy
```

## Índice completo de secciones

| # | Tema |
|---|---|
| 1 | [Cargar datos](docs/01-carga-e-inspeccion.md#1-cargar-datos) |
| 2 | [Inspección rápida](docs/01-carga-e-inspeccion.md#2-inspección-rápida) |
| 3 | [Seleccionar columnas](docs/01-carga-e-inspeccion.md#3-seleccionar-columnas) |
| 4 | [Renombrar, eliminar, reordenar](docs/01-carga-e-inspeccion.md#4-renombrar-eliminar-reordenar) |
| 5 | [Antes de limpiar: copia de seguridad](docs/01-carga-e-inspeccion.md#5-antes-de-limpiar-copia-de-seguridad) |
| 6 | [Diagnóstico de nulos](docs/02-nulos-y-duplicados.md#6-diagnóstico-de-nulos) |
| 7 | [Filtrar filas con condición](docs/02-nulos-y-duplicados.md#7-filtrar-filas-con-condición) |
| 8 | [Opción A — Eliminar nulos: `.dropna()`](docs/02-nulos-y-duplicados.md#8-opción-a-eliminar-nulos-dropna) |
| 9 | [Opción B — Imputar: `.fillna()`](docs/02-nulos-y-duplicados.md#9-opción-b-imputar-fillna) |
| 10 | [Validación final](docs/02-nulos-y-duplicados.md#10-validación-final) |
| 11 | [Diccionario de datos — antes de programar](docs/02-nulos-y-duplicados.md#11-diccionario-de-datos-antes-de-programar) |
| 12 | [Diccionarios de Python (la estructura, no el de datos)](docs/02-nulos-y-duplicados.md#12-diccionarios-de-python-la-estructura-no-el-de-datos) |
| 13 | [Eliminar duplicados](docs/02-nulos-y-duplicados.md#13-eliminar-duplicados) |
| 14 | [Filtrar filas irrelevantes (valores imposibles)](docs/02-nulos-y-duplicados.md#14-filtrar-filas-irrelevantes-valores-imposibles) |
| 15 | [Auditar y validar la limpieza](docs/02-nulos-y-duplicados.md#15-auditar-y-validar-la-limpieza) |
| 16 | [Estandarizar texto (normalizar strings)](docs/03-tipos-texto-y-fechas.md#16-estandarizar-texto-normalizar-strings) |
| 17 | [Tipos de datos (dtypes)](docs/03-tipos-texto-y-fechas.md#17-tipos-de-datos-dtypes) |
| 18 | [Convertir texto a números (flujo completo)](docs/03-tipos-texto-y-fechas.md#18-convertir-texto-a-números-flujo-completo) |
| 19 | [Convertir fechas a datetime](docs/03-tipos-texto-y-fechas.md#19-convertir-fechas-a-datetime) |
| 20 | [Regla general: conversiones seguras con `errors="coerce"`](docs/03-tipos-texto-y-fechas.md#20-regla-general-conversiones-seguras-con-errorscoerce) |
| 21 | [Filtrado avanzado de filas](docs/04-filtrado-y-estadistica.md#21-filtrado-avanzado-de-filas) |
| 22 | [Estadísticas resumidas (descriptivas)](docs/04-filtrado-y-estadistica.md#22-estadísticas-resumidas-descriptivas) |
| 23 | [Agrupar y agregar — `.groupby()`](docs/04-filtrado-y-estadistica.md#23-agrupar-y-agregar-groupby) |
| 24 | [Ordenar datos — `.sort_values()`](docs/04-filtrado-y-estadistica.md#24-ordenar-datos-sort_values) |
| 25 | [Preparar datasets para unirlos (pre-merge)](docs/05-merge-y-exportacion.md#25-preparar-datasets-para-unirlos-pre-merge) |
| 26 | [Validación visual (matplotlib + seaborn)](docs/05-merge-y-exportacion.md#26-validación-visual-matplotlib-seaborn) |
| 27 | [Combinar datasets — `pd.merge()`](docs/05-merge-y-exportacion.md#27-combinar-datasets-pdmerge) |
| 28 | [Organizar la tabla final (post-merge)](docs/05-merge-y-exportacion.md#28-organizar-la-tabla-final-post-merge) |
| 29 | [Exportar resultados — `.to_csv()`](docs/05-merge-y-exportacion.md#29-exportar-resultados-to_csv) |
| 30 | [Revisión inicial: roles, faltantes e inválidos (diagnóstico de calidad)](docs/06-calidad-de-datos.md#30-revisión-inicial-roles-faltantes-e-inválidos-diagnóstico-de-calidad) |
| 31 | [Manejar valores ausentes e inválidos (MCAR/MAR/MNAR + imputación)](docs/06-calidad-de-datos.md#31-manejar-valores-ausentes-e-inválidos-mcarmarmnar-imputación) |
| 32 | [Funciones para automatizar la limpieza (`def`)](docs/07-python-para-limpieza.md#32-funciones-para-automatizar-la-limpieza-def) |
| 33 | [Bucles (`for` & `while`) e iterables en una colección](docs/07-python-para-limpieza.md#33-bucles-for-while-e-iterables-en-una-colección) |
| 34 | [Mini-funciones de limpieza — un loop DENTRO de una función](docs/07-python-para-limpieza.md#34-mini-funciones-de-limpieza-un-loop-dentro-de-una-función) |
| 35 | [Pipeline completo de limpieza — de mini-funciones a `clean_data(df)`](docs/07-python-para-limpieza.md#35-pipeline-completo-de-limpieza-de-mini-funciones-a-clean_datadf) |
| 36 | [Diagnosticar la forma de la distribución (media y mediana sobre el histograma)](docs/08-distribuciones.md#36-diagnosticar-la-forma-de-la-distribución-media-y-mediana-sobre-el-histograma) |
| 37 | [Medidas descriptivas en columnas categóricas](docs/08-distribuciones.md#37-medidas-descriptivas-en-columnas-categóricas) |
| 38 | [Construir e interpretar histogramas (bins, bordes y escala)](docs/08-distribuciones.md#38-construir-e-interpretar-histogramas-bins-bordes-y-escala) |
| 39 | [Boxplots y detección de outliers (IQR)](docs/09-outliers.md#39-boxplots-y-detección-de-outliers-iqr) |
| 40 | [Interpretar la forma de una distribución en clave de negocio](docs/08-distribuciones.md#40-interpretar-la-forma-de-una-distribución-en-clave-de-negocio) |
| 41 | [Detectar outliers con reglas estadísticas — IQR y Z-score](docs/09-outliers.md#41-detectar-outliers-con-reglas-estadísticas-iqr-y-z-score) |
| 42 | [Qué hacer con un outlier — KEEP, DROP y CAP (winsorización)](docs/09-outliers.md#42-qué-hacer-con-un-outlier-keep-drop-y-cap-winsorización) |
| 43 | [Lógica condicional — `if` / `else`](docs/07-python-para-limpieza.md#43-lógica-condicional-if-else) |
| 44 | [Feature engineering — crear una columna de segmento](docs/10-feature-engineering.md#44-feature-engineering-crear-una-columna-de-segmento) |
| 45 | [Comparar una distribución entre grupos (`hue` en histplot)](docs/08-distribuciones.md#45-comparar-una-distribución-entre-grupos-hue-en-histplot) |

## Licencia

[MIT](LICENSE). Úsala, cópiala, adáptala.
