# Filtrado y estadística descriptiva

[← Índice de la guía](../README.md)

---

## 21. Filtrado avanzado de filas

**Cuándo usarlo:** después de limpiar, antes de calcular promedios o graficar. Aísla los registros que responden la pregunta de negocio (ej. solo países X con valores altos y fuente confiable).

**Estructura base — siempre la misma:** `df[ condición ]`, donde la condición es cualquier código que devuelva True/False.

### 🔁 Sintaxis genérica (copiar y adaptar)

```python
# Numérica: filas donde 'nombre_columna' supera un valor
filtro = df[df["nombre_columna"] > valor]

# Texto: filas donde 'nombre_columna' contiene "ejemplo" (sin distinguir mayúsculas)
filtro = df[df["nombre_columna"].str.contains("ejemplo", case=False)]

# Pertenencia: filas donde 'nombre_columna' está en la lista
filtro = df[df["nombre_columna"].isin(["valor1", "valor2"])]

# Combinadas: cada condición entre paréntesis
filtro = df[(df["col_1"] > valor) & (df["col_2"].isin(["a", "b"]))]
```

### Condiciones numéricas

```python
df_high  = df[df["value"] > 50]     # mayores que
df_low   = df[df["value"] <= 25]    # menores o iguales
df_equal = df[df["value"] == 50]    # exactamente iguales (doble ==)
```

| Operador | Significado |
|---|---|
| `>` / `>=` | mayor / mayor o igual |
| `<` / `<=` | menor / menor o igual |
| `==` | igual a (⚠️ doble signo — uno solo es asignación) |

Truco: el lado ABIERTO del símbolo apunta a lo que esperas mayor.

### Condiciones de texto — `.str.contains()`

```python
df_owner = df[df["owner_name"].str.contains("government", case=False)]
```

- `case=False` → ignora mayúsculas/minúsculas ("Government", "GOVERNMENT", "government" cuentan igual).
- **Cuándo:** buscar palabras o patrones dentro de texto no estandarizado.

### Condición de pertenencia — `.isin()`

```python
# Directo
df_asia = df[df["country_id"].isin(["KOR", "IND", "IDN"])]

# Con lista separada (mejor para listas largas)
paises = ["KOR", "IND", "IDN"]
df_asia = df[df["country_id"].isin(paises)]
```

- `.isin()` = "está en" la lista.
- **Cuándo:** conservar varios valores específicos sin escribir múltiples comparaciones con `|`.

### Combinar condiciones — `&` (Y) y `|` (O)

```python
# TODAS las condiciones a la vez → & (resultado: pocas filas)
df_target = df[
    (df["value"] > 50) &
    (df["owner_name"].str.contains("government", case=False)) &
    (df["country_id"].isin(["KOR", "IND", "IDN"]))
]

# AL MENOS UNA condición → | (resultado: muchas filas)
df_target = df[
    (condicion1) | (condicion2) | (condicion3)
]
```

**Reglas críticas:**

- ⚠️ Cada condición VA ENTRE PARÉNTESIS — sin ellos, pandas lanza error o evalúa mal.
- `&` restringe (todas se cumplen) → menos filas. `|` amplía (basta una) → más filas. Ejemplo real: sobre el mismo dataset, `&` dio 27 registros y `|` dio 4,741.
- Guarda el resultado en una variable nueva (`df_target = ...`) para no pisar tu dataset limpio.

### ¿Qué condición para qué pregunta?

| Pregunta | Herramienta |   |
|---|---|---|
| ¿Valores arriba/abajo de un umbral? | `>`, `<`, `>=`, `<=`, `==` |   |
| ¿El texto contiene una palabra/patrón? | `.str.contains("palabra", case=False)` |   |
| ¿El valor está en mi lista de interés? | `.isin([...])` |   |
| ¿Se cumplen varias a la vez? | `(cond1) & (cond2)` |   |
| ¿Basta con que se cumpla una? → operador O (barra vertical), ver bloque de sintaxis genérica | \`(cond1) \\ | (cond2)\` |

---

## 22. Estadísticas resumidas (descriptivas)

**Cuándo usarlo:** después de limpiar y filtrar. Resumen miles de registros en pocos números interpretables — la base de dashboards e informes.

### Las 5 medidas y sus funciones

| Grupo | Medida | Función | Qué te dice |
|---|---|---|---|
| Tendencia central | Media (promedio) | `.mean()` | Valor típico — sensible a extremos |
| Tendencia central | Mediana | `.median()` | Valor central — robusta a extremos |
| Extremos | Mínimo | `.min()` | El valor más bajo registrado |
| Extremos | Máximo | `.max()` | El valor más alto registrado |
| Dispersión | Desviación estándar | `.std()` | Qué tanto se alejan los datos del promedio |

### 🔁 Sintaxis genérica (copiar y adaptar)

```python
# Estadísticas de una columna numérica
df["nombre_columna"].mean()      # promedio
df["nombre_columna"].median()    # mediana
df["nombre_columna"].min()       # mínimo
df["nombre_columna"].max()       # máximo
df["nombre_columna"].std()       # desviación estándar

# Estadísticas de un subgrupo (filtrar primero, calcular después)
df_grupo = df[df["columna_categoria"] == "valor"]
df_grupo["nombre_columna"].mean()
```

### ¿Media o mediana? La decisión clave

- **Media** → datos estables, sin valores extremos.
- **Mediana** → hay outliers; no se ve afectada por ellos.
- ⚠️ **Señal de alerta:** si media y mediana son MUY distintas (ej. media 57 vs mediana 8), hay valores extremos inflando el promedio → usa la mediana como valor típico y revisa los outliers.

### Cómo interpretar en conjunto (ejemplo real: México, PM2.5)

| Medida | Valor | Lectura |
|---|---|---|
| Media | 186.33 | Nivel promedio |
| Mediana | 16.56 | Mucho menor que la media → hay extremos altos |
| Mínimo | 0.0 | Momentos de aire limpio |
| Máximo | 4,756.54 | Picos anómalos o eventos puntuales |
| Desv. estándar | 792.41 | Alta dispersión → mediciones muy variables |

**Conclusión tipo:** "La mediana es el valor típico confiable; la media está inflada por picos. El rango min-max señala posibles anomalías y la std confirma alta variabilidad."

### Interpretación de la desviación estándar

- **Baja** (ej. Canadá: std ≈ 2) → datos estables, concentrados cerca del promedio.
- **Alta** (ej. USA: std ≈ 220) → datos muy variables, con valores lejanos al promedio.

### Comparar entre grupos (patrón país por país)

```python
df_can = df_clean[df_clean["country_id"] == "CAN"]
df_usa = df_clean[df_clean["country_id"] == "USA"]

df_can["value"].std()    # ≈ 2 → estable
df_usa["value"].std()    # ≈ 220 → muy variable
```

Mismo esquema para `.min()`, `.max()`, `.mean()`, `.median()` — filtras el grupo y calculas. Comparar la misma medida entre grupos revela dónde está el problema (ej. USA tiene picos de 3,473 vs máximo de 12.5 en Canadá).

📌 **Conecta con las secciones 26 y 36.** Estas medidas cobran sentido cuando las **ves** sobre la distribución: la 36.1 dibuja media y mediana encima del histograma con `plt.axvline()`, y la 36.4 muestra el caso límite de esta misma comparación entre grupos — tres clases con la **misma media** que solo se distinguen por `std` y el rango mín–máx.

---

## 23. Agrupar y agregar — `.groupby()`

**Cuándo usarlo:** cuando un promedio general no basta y necesitas comparar entre categorías (por país, ciudad, mes, producto...). Es la tabla dinámica de Excel o el `GROUP BY` de SQL, pero en pandas.

**Los dos conceptos:**

- **Agrupar** = dividir los datos en subconjuntos (una "cajita" por cada categoría) → `.groupby()`
- **Agregar** = calcular un resumen dentro de cada grupo → `.mean()`, `.sum()`, `.count()`

⚠️ `.groupby()` solo NO devuelve nada útil — siempre necesita una función de agregación después.

### La estructura (memorízala como plantilla)

```python
# Básica: agrupar → aplicar función
nombre_df.groupby("columna_grupo").funcion()

# Enfocada a UNA columna (lo más común)
nombre_df.groupby("columna_grupo")["columna_valor"].funcion()

# Dos o más columnas de valor
nombre_df.groupby("columna_grupo")[["col1", "col2"]].funcion()
```

### 🔁 Sintaxis genérica (copiar y adaptar)

```python
# Promedio por grupo
df.groupby("columna_grupo")["columna_valor"].mean().reset_index()

# Conteo de registros por grupo (¿cuánta cobertura de datos hay?)
df.groupby("columna_grupo")["columna_valor"].count().reset_index()

# Suma acumulada por grupo
df.groupby("columna_grupo")["columna_valor"].sum().reset_index()

# Ordenar el resultado (ascending=False → de mayor a menor)
df.groupby("columna_grupo")["columna_valor"].mean()\
    .reset_index()\
    .sort_values(by="columna_valor", ascending=False)
```

### Qué función para qué pregunta

| Pregunta | Función |
|---|---|
| ¿Cuál es el nivel típico por grupo? | `.mean()` |
| ¿Cuántas mediciones/registros tiene cada grupo? | `.count()` |
| ¿Cuál es el total acumulado por grupo? | `.sum()` |
| ¿Varias medidas a la vez? | `.agg([...])` |

### Los acompañantes de la cadena

- `.reset_index()` → convierte el resultado agrupado en un DataFrame legible y usable.
- `.sort_values(by="col")` → ordena de menor a mayor; `ascending=False` invierte a mayor→menor.
- `\` al final de línea → salto de línea para que la cadena larga sea legible.

### Varias medidas a la vez — `.agg()`

```python
# El resumen completo en una línea: promedio + total + conteo por grupo
df.groupby("columna_grupo")["columna_valor"]\
    .agg(["mean", "sum", "count"])\
    .reset_index()\
    .sort_values(by="mean", ascending=False)\
    .rename(columns={"mean": "promedio", "sum": "total_acumulado", "count": "registros"})
```

reset_index()

- Las funciones van como **lista de strings**: `["mean", "sum", "count"]`.
- `.rename(columns={...})` al final traduce los nombres técnicos a etiquetas claras (mismo diccionario de la sección 4).
- `by=` en el sort usa el nombre ORIGINAL de la función (`"mean"`) si el rename va después.

### Interpretación tipo analista

- **count alto + mean bajo** → mucha cobertura, niveles bajos: dato confiable.
- **mean alto + count bajo** (ej. un país con 1 registro) → ⚠️ el promedio puede no ser representativo; revisa antes de concluir.
- Comparar `count` entre grupos revela vacíos de datos y qué tan defendible es cada comparación.

---

## 24. Ordenar datos — `.sort_values()`

**Cuándo usarlo:** para identificar patrones y extremos: rankings (top 10), quién destaca por arriba o por abajo, y para preparar tablas legibles para reportes. Se puede aplicar antes de agrupar (dataset completo), después de filtrar (subconjunto) o después de agrupar (resultados agregados).

### 🔁 Sintaxis genérica (copiar y adaptar)

```python
# Ordenar por UNA columna (menor a mayor — default)
df.sort_values(by="nombre_columna")

# De mayor a menor
df.sort_values(by="nombre_columna", ascending=False)

# Top 10 más bajos / más altos
df.sort_values(by="nombre_columna", ascending=True).head(10)
df.sort_values(by="nombre_columna", ascending=False).head(10)

# Por VARIAS columnas (criterio principal + desempate)
df.sort_values(by=["columna_1", "columna_2"], ascending=[False, True])
```

### Parámetros clave

| Parámetro | Valor | Efecto |
|---|---|---|
| `by=` | `"columna"` o `["col1", "col2"]` | Por qué columna(s) ordenar |
| `ascending=True` | (default) | Menor → mayor |
| `ascending=False` |   | Mayor → menor |
| `ascending=[False, True]` | lista | Una dirección POR CADA columna de `by`, en el mismo orden |

⚠️ Con varias columnas: el primer valor de `ascending` aplica a la primera columna de `by`, el segundo a la segunda, etc.

### Ordenar por múltiples columnas (criterio jerárquico)

**Cuándo:** cuando una sola columna no basta y necesitas desempatar (ej. ordenar por cantidad de registros, y en empates, alfabéticamente).

```python
df_country.sort_values(
    by=["registros", "country_name"],     # 1º registros, 2º desempate alfabético
    ascending=[False, True]               # registros: mayor→menor; nombre: A→Z
)
```

### El patrón "ranking" completo (groupby + sort + head)

```python
# Top 10 de grupos con promedio más alto
df_country = df.groupby("columna_grupo")["columna_valor"]\
    .mean()\
    .reset_index()

df_country.sort_values(by="columna_valor", ascending=False).head(10)   # los 10 más altos
df_country.sort_values(by="columna_valor", ascending=True).head(10)    # los 10 más bajos
```

**Truco:** para el "top 10 más bajos" NO necesitas `tail()` — cambia `ascending` y usa `head(10)`.

### Dónde aplicar el orden según la etapa

| Etapa | Código tipo |
|---|---|
| Dataset completo | `df.sort_values(by="col")` |
| Después de filtrar | `df[df["pais"] == "X"].sort_values(by="col")` |
| Después de agrupar | `df.groupby("g")["v"].mean().reset_index().sort_values(by="v")` |

---

[← Tipos de datos, texto y fechas](03-tipos-texto-y-fechas.md) · [Índice](../README.md) · [Combinar datasets y exportar →](05-merge-y-exportacion.md)
