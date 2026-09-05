# Combinar datasets y exportar

[← Índice de la guía](../README.md)

---

## 25. Preparar datasets para unirlos (pre-merge)

**Cuándo usarlo:** SIEMPRE antes de combinar dos tablas (`merge`). Es el `JOIN` de SQL en pandas — pero si las claves no están limpias y compatibles, la unión falla o produce basura silenciosamente.

**Clave de unión (key):** columna(s) que identifican unívocamente cada observación y sirven de "puente" entre tablas (ej. país + año). Analogía: el número de identidad de una persona — mismo ID en ambas bases = mismo individuo.

### Checklist pre-merge (los 5 chequeos)

| # | Chequeo | Herramienta |
|---|---|---|
| 1 | ¿Qué columnas tiene cada tabla? ¿Cuáles comparten significado? | `.columns` |
| 2 | ¿Nombres y formatos compatibles? | `.rename()`  • `.str.title()` |
| 3 | ¿La clave es única? (sin duplicados) | `.duplicated().sum()` → 0 |
| 4 | ¿Cardinalidad: los valores existen en AMBAS tablas? | `.nunique()`  • `.value_counts()` |
| 5 | ¿Mismo tipo de dato en las claves? | `.dtypes`  • `.astype(int)` |

### 🔁 Sintaxis genérica (copiar y adaptar)

```python
# 1) Explorar columnas de ambas tablas
print(df_a.columns)
print(df_b.columns)

# 2) Normalizar nombres y formato (mismo nombre + mismo casing en ambas)
df_b = df_b.rename(columns={"nombre_distinto": "nombre_comun"})
df_a["nombre_comun"] = df_a["nombre_comun"].str.title()
df_b["nombre_comun"] = df_b["nombre_comun"].str.title()

# 3) Verificar unicidad de la clave (por tabla)
df_a[["clave_1", "clave_2"]].duplicated().sum()   # debe dar 0

# 4) Cardinalidad: ¿coinciden las categorías y periodos?
df_a["clave_1"].nunique(), df_b["clave_1"].nunique()
df_a["clave_2"].value_counts().sort_index()       # distribución por periodo

# 5) Mismo tipo en las claves
print(df_a["clave_2"].dtypes)
print(df_b["clave_2"].dtypes)
df_a["clave_2"] = df_a["clave_2"].astype(int)     # convertir si difieren
```

### Nuevo: `.str.title()` — formato título

```python
df["nombre_columna"] = df["nombre_columna"].str.title()   # "chile"/"CHILE" → "Chile"
```

Se suma a `.str.upper()` / `.str.lower()` (sección 16). Úsalo cuando la misma categoría aparece con distinto casing en cada tabla — sin esto, `"Chile"` ≠ `"chile"` y el merge no encuentra coincidencias.

### Si la clave tiene duplicados: agregar, no borrar

Múltiples mediciones del mismo país-año (ej. varios sensores) NO se eliminan — se **colapsan a una fila por clave** con el promedio:

```python
group_cols = ["clave_1", "clave_2"]
df_agg = df.groupby(group_cols, dropna=False).agg({"columna_valor": "mean"}).reset_index()
```

- `.agg({"col": "mean"})` con **diccionario**: qué columna → qué función.
- `dropna=False` → no descarta grupos con claves nulas (los mantiene visibles).
- Resultado: una fila por clave → merge seguro sin inflar resultados.

### Nuevo: `.sort_index()` y extraer año para la clave

```python
# value_counts ordenado por índice (años cronológicos, no por frecuencia)
df["anio"].value_counts().sort_index()

# Crear la clave de año desde una fecha completa (cuando una tabla tiene fecha y la otra solo año)
df["anio"] = pd.to_datetime(df["col_fecha"]).dt.year
```

### Interpretación tipo analista

- `.duplicated().sum()` \> 0 en la clave → varios registros por clave: agrega con groupby antes de unir, o el merge **multiplicará filas**.
- `nunique` distinto entre tablas (ej. 2 vs 3 países) → habrá registros sin pareja: decide qué tipo de join usar y documenta la pérdida.
- `value_counts` por año revela cobertura desigual (ej. una tabla con 1990–2021, la otra solo 2021) → filtra al rango común antes de unir.
- Tipos distintos en la clave (int vs texto) → el merge falla o une 0 filas. Convierte primero.

---

## 26. Validación visual (matplotlib + seaborn)

**Cuándo usarlo:** antes de unir o analizar. Los números (`.info()`, `.describe()`) pueden verse bien y aun así esconder problemas — un gráfico revela de inmediato categorías duplicadas ("México" vs "Mexico"), años faltantes o valores imposibles.

### Los imports estándar

```python
import pandas as pd
import matplotlib.pyplot as plt   # plt = alias estándar de matplotlib
import seaborn as sns             # sns = alias estándar de seaborn
```

### 🔁 Sintaxis genérica (copiar y adaptar)

```python
# BARRAS — frecuencia de categorías (¿hay duplicados o desbalance?)
df["nombre_columna"].value_counts().plot(kind="bar", figsize=(10, 5))
plt.title("Título del gráfico")
plt.xlabel("Eje X")
plt.ylabel("Conteo")
plt.show()

# BARRAS por periodo — ordenar cronológicamente ANTES de graficar
df["anio"].value_counts().sort_index().plot(kind="bar")
plt.show()

# BOXPLOT — detectar outliers en una variable numérica
# showmeans=True dibuja también la MEDIA (además de la mediana de la caja)
sns.boxplot(data=df, x="columna_numerica", showmeans=True)
plt.title("Distribución")
plt.show()

# BOXPLOT por categoría — comparar distribución entre grupos
sns.boxplot(data=df, x="columna_numerica", y="columna_categoria")
plt.show()

# HISTOGRAMA — forma de la distribución
df["columna_numerica"].hist(bins=5, figsize=(10, 5))   # bins = nº de barras
plt.show()

# COUNTPLOT — value_counts() graficado directo, ordenado de mayor a menor
sns.countplot(data=df, y="columna_categoria",
              order=df["columna_categoria"].value_counts().index)
plt.show()
```

### Qué gráfico para qué validación

| Pregunta de validación | Gráfico | Código base |
|---|---|---|
| ¿Categorías duplicadas o desbalanceadas? | Barras / countplot | `value_counts().plot(kind="bar")` o `sns.countplot()` |
| ¿Años completos, sin huecos ni duplicados? | Barras ordenadas | `value_counts().sort_index().plot(kind="bar")` |
| ¿Valores atípicos (outliers)? | Boxplot | `sns.boxplot(data=df, x="col")` |
| ¿Forma de la distribución? | Histograma | `df["col"].hist(bins=n)` |

### Anatomía del boxplot (diagrama de caja)

- **Caja** = del Q1 (25%) al Q3 (75%); la línea del centro es la **mediana** (Q2).
- **Bigotes** = rango de valores "normales" (mín y máx sin outliers).
- **Puntos fuera de los bigotes** = valores atípicos → investigar: ¿error de captura o evento real?
- Intercambiar `x=` y `y=` rota el gráfico (horizontal ↔ vertical).

### Parámetros de personalización

| Parámetro / función | Qué hace |
|---|---|
| `kind="bar"` | Tipo de gráfico en `.plot()` |
| `figsize=(ancho, alto)` | Tamaño del gráfico |
| `plt.title("...")` | Título |
| `plt.xlabel("...")` / `plt.ylabel("...")` | Etiquetas de ejes |
| `plt.show()` | Muestra el gráfico (cierra la configuración — siempre al final) |
| `bins=n` | Número de barras del histograma |
| `order=...value_counts().index` | Ordena countplot de mayor a menor frecuencia |

### Eje Y secundario — `secondary_y` (dos escalas en un gráfico)

**Cuándo usarlo:** cuando comparas en un mismo gráfico dos variables con rangos muy distintos (ej. tráfico en cientos vs PIB en decenas de miles). Señal de alerta: una de las series se ve "aplastada contra el cero" — su escala quedó dominada por la otra.

**Cómo funciona:** la columna que indiques en `secondary_y` se dibuja con su propia escala en el eje DERECHO; el resto usa el eje izquierdo. Así cada variable se lee en su rango natural.

**🔁 Sintaxis genérica:**

```python
# Dos variables de escalas distintas en un gráfico de barras
df.plot(x="columna_categoria", y=["columna_chica", "columna_grande"], kind="bar",
        secondary_y="columna_grande", figsize=(12, 5))
plt.title("Título")
plt.xticks(rotation=90)   # rota las etiquetas del eje X si son largas
plt.show()
```

⚠️ Sin el eje secundario, la comparación visual entre escalas distintas es engañosa — fue la única corrección del proyecto S5.

### Matplotlib vs Seaborn (equivalencias)

| Tipo | Matplotlib | Seaborn |
|---|---|---|
| Barras | `df.plot(kind="bar")` | `sns.barplot(x=, y=)` |
| Boxplot | `plt.boxplot(df)` | `sns.boxplot(data=, x=)` |
| Histograma | `df["col"].hist()` | `sns.histplot(df)` |
| Countplot | no directo (bar + value_counts) | `sns.countplot(data=, y=)` |

### Interpretación tipo analista

- Un salto en la distribución por años (ej. datos de 1990 y luego nada hasta 2019) → ¿faltan datos o error de carga? Documéntalo.
- Dos barras casi idénticas con nombres similares → misma categoría escrita distinto: vuelve a la sección 16 (estandarizar texto).
- Outliers en el boxplot no se borran automáticamente → primero pregunta si son plausibles (sección 14).

📌 **Conecta con las secciones 22 y 36.** El histograma de esta sección (`df["col"].hist()`) muestra la forma, pero sin puntos de referencia. En la 36.1 le añades las líneas de media y mediana con `plt.axvline()` para diagnosticar el sesgo de un vistazo, en la 36.2 está la trampa de `range=` (recorta el gráfico, no los datos) y en la 36.6 generas un histograma por grupo dentro de un bucle.

---

## 27. Combinar datasets — `pd.merge()`

**Cuándo usarlo:** cuando la información que necesitas está repartida en dos tablas (ej. contaminación + salud) y comparten una clave (país + año). Es el `JOIN` de SQL en pandas. Requiere la preparación de la sección 25 hecha ANTES.

### 🔁 Sintaxis genérica (copiar y adaptar)

```python
# Estructura: merge(tabla_izquierda, tabla_derecha, on=claves, how=tipo)
df_unido = pd.merge(df_izq, df_der, on=["clave_1", "clave_2"], how="inner")

# Left join: conserva TODO el df de la izquierda
df_unido = pd.merge(df_principal, df_extra, on=["clave_1", "clave_2"], how="left")
```

### Los 4 tipos de unión (`how=`)

| Tipo | Qué conserva | Cuándo usarlo |
|---|---|---|
| `inner` | Solo filas con clave en AMBAS tablas | Solo quieres registros completos (intersección) |
| `left` | TODAS las filas de la izquierda + coincidencias de la derecha | Mantener tu tabla principal intacta y "agregarle" datos extra |
| `right` | TODAS las filas de la derecha + coincidencias de la izquierda | Igual que left pero al revés |
| `outer` | TODAS las filas de ambas, rellenando con NaN | Dataset completo sin perder nada |

💡 Los más usados: `inner` y `left`. En un `left`, las filas sin pareja quedan con `NaN` en las columnas de la derecha — eso NO es error, es información: te dice dónde faltan datos.

### Detalle clave: columnas repetidas → sufijos `_x` y `_y`

Si ambas tablas tienen una columna con el mismo nombre (ej. `value`) que NO es clave, el resultado trae `value_x` (de la tabla izquierda) y `value_y` (de la derecha). Identifica cuál es cuál por el ORDEN en que pasaste las tablas.

### Validación post-merge (los 4 chequeos)

```python
# 1) Conteo de filas antes vs después — ¿cuánto solapamiento hubo?
print("Tabla A:", len(df_a))
print("Tabla B:", len(df_b))
print("Inner:", len(df_inner))
print("Left:", len(df_left))

# 2) Duplicados inesperados — debe dar 0
df_unido[["clave_1", "clave_2"]].duplicated().sum()

# 3) Nulos generados por el merge (en left/outer)
df_unido.isna().sum()

# 4) Estructura y tipos intactos
df_unido.info()
df_unido.describe()
```

**Interpretación:**

- `inner` con MUCHAS menos filas que los originales → poco solapamiento entre fuentes; documenta qué se perdió.
- Duplicados \> 0 tras el merge → la clave no era única en alguna tabla y el merge **multiplicó filas** → vuelve a la sección 25 y agrega con groupby.
- Nulos concentrados en las columnas de una tabla → esas filas no encontraron pareja. Si superan el 30% de una variable, revísala antes de usarla.

### Detectar qué registros no se unieron — `set()`

```python
# Valores de la clave que están en una tabla pero NO llegaron al merge
faltantes = set(df_original["clave_1"]) - set(df_inner["clave_1"])
print("Sin pareja:", faltantes)
```

`set()` crea una colección sin duplicados; restar dos sets da lo que está en el primero y no en el segundo. Úsalo para decidir: ¿recolectar más datos o limitar el análisis a los registros completos?

### Flujo completo merge (resumen)

1. Preparar claves (sección 25: normalizar, deduplicar, tipos).
2. Elegir el `how` según la pregunta de negocio.
3. `pd.merge(izq, der, on=claves, how=tipo)`.
4. Validar: filas, duplicados, nulos, tipos.
5. Detectar discrepancias con `set()` y documentar.

---

## 28. Organizar la tabla final (post-merge)

**Cuándo usarlo:** después del merge, antes de analizar o compartir. Nombres claros + columnas relevantes + orden lógico + índice limpio = tabla profesional. Complementa la sección 4 con lo nuevo.

### Renombrar: `rename()` vs `.columns` — cuál elegir

| Método | Cómo funciona | Cuándo conviene |
|---|---|---|
| `.rename(columns={...})` | Solo mencionas las que cambian | Cambiar POCOS nombres |
| `.columns = [lista]` | Debes listar TODAS las columnas, en orden | Cambiar TODOS (o casi todos) los nombres |

```python
# Opción A — rename: solo las que cambian (ej. sufijos _x/_y del merge)
df = df.rename(columns={
    "value_x": "muertes_prom",
    "value_y": "contaminacion",
    "year_id": "anio"
})

# Opción B — .columns: lista completa obligatoria
df.columns = ["col_1", "col_2", "col_3"]   # tantas como tenga el DF
```

💡 Tras un merge, renombrar `value_x`/`value_y` a nombres descriptivos es casi obligatorio.

### Tips para nombres de columnas

Minúsculas, sin espacios (usa `_`), sin caracteres especiales, descriptivo pero corto, y el MISMO estilo en todo el proyecto.

### Nuevo: `drop()` con `errors="ignore"`

```python
df = df.drop(columns=["columna_1", "columna_2"], errors="ignore")
```

- `columns=[...]` — alternativa a `axis=1` de la sección 4, más legible.
- `errors="ignore"` → si alguna columna no existe, no lanza error (útil en scripts que se re-ejecutan).
- Qué eliminar: columnas redundantes (mismo valor en todas las filas) o que no aportan al análisis.

### Reordenar + reindexar (cierre estándar)

```python
# Reorden: identificadores primero, métricas después — mencionar TODAS las que conservas
df = df[["anio", "pais", "metrica_principal", "metrica_2", "detalle_1"]]

# Índice limpio para guardar/compartir
df = df.reset_index(drop=True)

# Validación final
df.info()
df.head()
```

Sugerencia de orden: variables de identificación al inicio (país, año), luego las métricas principales, al final los detalles.

### Reproducibilidad (práctica profesional)

- Guarda las transformaciones en un script/notebook documentado — el análisis debe poder repetirse con los datos del próximo mes sin reescribir nada.
- Comenta el **por qué** de cada cambio (no solo el qué).
- Conserva siempre el dataset original intacto (tu regla de `df.copy()`, sección 5).
- Separa los pasos de limpieza de los pasos de análisis.

---

## 29. Exportar resultados — `.to_csv()`

**Cuándo usarlo:** al final del pipeline, cuando el dataset ya está limpio, unido y validado. Es la contraparte de `pd.read_csv()`.

### 🔁 Sintaxis genérica (copiar y adaptar)

```python
# Exportar a CSV sin la columna de índice
df.to_csv("nombre_archivo_limpio.csv", index=False)
```

- `index=False` → no guarda el índice (0, 1, 2...) como columna extra. Casi siempre lo quieres así.
- Antes de exportar: `reset_index(drop=True)` + validación final (`info()`, `isna().sum()`) — secciones 15 y 28.

### Tips de nombre de archivo

- Descriptivo y en snake_case: `ladb_mobility_economy_2024_clean.csv`
- Incluye el periodo (`2024`) y el estado (`clean`) — tu "yo" del futuro lo agradecerá.
- Nunca sobrescribas el archivo original crudo.

---

[← Filtrado y estadística descriptiva](04-filtrado-y-estadistica.md) · [Índice](../README.md) · [Diagnóstico de calidad e imputación →](06-calidad-de-datos.md)
