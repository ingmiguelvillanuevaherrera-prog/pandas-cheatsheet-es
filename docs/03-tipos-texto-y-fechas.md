# Tipos de datos, texto y fechas

[← Índice de la guía](../README.md)

---

## 16. Estandarizar texto (normalizar strings)

**Cuándo usarlo:** antes de agrupar o calcular KPIs por categoría. Si una columna mezcla `"Arg "`, `"argentina"` y `"ARG"`, pandas las cuenta como categorías DISTINTAS y fragmenta tus métricas.

### Los dos limpiadores básicos

```python
# Quitar espacios al inicio/final + unificar mayúsculas (en una sola línea)
df["country_id"] = df["country_id"].str.strip().str.upper()

# Variantes
.str.upper()    # TODO A MAYÚSCULAS
.str.lower()    # todo a minúsculas
.str.strip()    # quita espacios a los lados (" PER" → "PER")

# Asegurar tipo string antes de operar (opcional pero robusto)
df["country_id"] = df["country_id"].astype("string").str.upper()
```

⚠️ Guarda el cambio en la misma columna (`df["col"] = ...`) o se pierde.

### Detectar el problema antes de limpiar

```python
# Explorar valores únicos (sin guardar nada — solo exploración)
df["country_id"].dropna().unique()
df["country_id"].sort_values().dropna().unique()   # ordenados alfabéticamente

# Filtrar filas que contienen un espacio (los espacios no se VEN, pero están)
df[df["country_id"].str.contains(" ")]["country_id"]
```

**Flujo de validación:** filtrar con `.str.contains(" ")` → aplicar `.str.strip()` → volver a filtrar → debe devolver VACÍO.

**Por qué importa:** un espacio invisible duplica la categoría (`"PER"` ≠ `" PER"`) → doble conteo → KPIs por país distorsionados.

### 🔁 Sintaxis genérica (copiar y adaptar)

```python
# Limpia texto: quita espacios y unifica mayúsculas, guardando en la misma columna
df["nombre_columna"] = df["nombre_columna"].str.strip().str.upper()

# Filtra filas donde 'nombre_columna' contiene un patrón
filtro = df[df["nombre_columna"].str.contains("ejemplo")]
```

---

## 17. Tipos de datos (dtypes)

**Cuándo usarlo:** chequeo inicial al cargar datos y validación final después de limpiar/convertir.

```python
print(df.dtypes)        # tipo de cada columna
df["col"].dtype         # tipo de UNA columna
```

### Dtypes más comunes

| dtype | Qué es | Ejemplo |
|---|---|---|
| `int64` | Enteros | Conteos, IDs numéricos |
| `float64` | Decimales | Medidas, precios |
| `object` | Texto genérico o tipos mezclados | ⚠️ Problemático si esperabas números |
| `datetime64[ns]` | Fechas/horas | Habilita filtros por rango, extraer año/mes |
| `bool` | Verdadero/falso | Banderas |
| `category` | Texto con pocos valores únicos | Ahorra memoria, acelera groupby |

### Regla de alerta

Si ves **`object` donde esperas números o fechas → hay que convertir**. `object` bloquea cálculos (promedios, min/max) y esconde errores: espacios, comas decimales, `"NULL"`, etc.

| Conversión | Herramienta |
|---|---|
| object → número | `pd.to_numeric(df[col], errors="coerce")` |
| object → fecha | `pd.to_datetime(df[col], errors="coerce")` |
| Cualquier tipo explícito | `df[col].astype(tipo)` |

`errors="coerce"` → lo que no se puede convertir se vuelve `NaN` (después lo atrapas con tu flujo de nulos).

---

## 18. Convertir texto a números (flujo completo)

**Cuándo usarlo:** cuando `.dtype` da `object` en una columna que debería ser numérica. Sin conversión no hay promedios, min/max ni comparativas — y el ordenamiento es alfabético (`"10" < "9"`).

### Paso 1 — Diagnosticar ANTES de convertir

```python
df["unit_2"].dtype     # confirma que es object

# ¿Qué valores fallarían al convertir? (dimensionar el daño)
mask_non_numeric = pd.to_numeric(df["unit_2"], errors="coerce")
df[mask_non_numeric.isna()]["unit_2"]     # ver la "basura": comas, espacios, guiones
```

**Truco:** conviertes a una variable temporal (sin guardar en la columna) y filtras los que quedaron `NaN` — esos son los problemáticos.

### Paso 2 — Limpiar y convertir (secuencia encadenada)

```python
df["unit_2"] = (
      df["unit_2"]             # 1. seleccionar la columna
    .astype("string")          # 2. garantizar tipo texto
    .str.strip()               # 3. quitar espacios extremos
    .str.replace(",", ".")     # 4. unificar decimales: coma → punto
)

df["unit_2"] = pd.to_numeric(df["unit_2"], errors="coerce")   # 5. convertir y guardar
```

`.str.replace(qué_busco, con_qué_reemplazo)` — primero el patrón, luego el reemplazo.

### Paso 3 — Validar

```python
df["unit_2"].dtype          # debe ser float64 (u otro numérico)
df["unit_2"].isna().sum()   # cuántos NaN dejó la conversión
df["unit_2"].describe()     # ¿las estadísticas tienen sentido?
```

Los `NaN` tras la conversión **no son un fallo** — son evidencia auditable. Después decides: imputar (mediana/constante) o filtrar, con tu flujo de nulos (sección 9).

### 🔁 Sintaxis genérica (copiar y adaptar)

```python
# Limpia el texto de 'nombre_columna' y la convierte a número de forma segura
df["nombre_columna"] = (
    df["nombre_columna"]
    .astype("string")
    .str.strip()
    .str.replace(",", ".")
)
df["nombre_columna"] = pd.to_numeric(df["nombre_columna"], errors="coerce")
```

---

## 19. Convertir fechas a datetime

**Cuándo usarlo:** cuando la columna temporal está como texto. Con texto, `"2021-12"` aparece antes que `"2021-2"` al ordenar. Con `datetime` real puedes ordenar cronológicamente, filtrar por rangos y extraer año/mes.

### Diagnosticar → convertir → validar

```python
# 1) DIAGNÓSTICO
df["lastUpdated"].dtype
prueba = pd.to_datetime(df["lastUpdated"], errors="coerce", utc=True)
df[prueba.isna()]["lastUpdated"]      # valores que fallan el parseo

# 2) CONVERSIÓN segura (guardando)
df["lastUpdated"] = pd.to_datetime(df["lastUpdated"], errors="coerce", utc=True)

# 3) VALIDACIÓN
df["lastUpdated"].dtype               # datetime64[ns, UTC]
df["lastUpdated"].isna().sum()        # ¿cuántas quedaron como NaT?
```

- `errors="coerce"` → fechas inválidas se vuelven **`NaT`** (Not a Time), el NaN de las fechas.
- `utc=True` → normaliza zona horaria; clave al combinar datos de varias regiones.

### 🔁 Sintaxis genérica (copiar y adaptar)

```python
# Convierte 'nombre_columna' a fecha de forma segura (inválidos → NaT)
df["nombre_columna"] = pd.to_datetime(df["nombre_columna"], errors="coerce", utc=True)

# Extrae componentes en columnas nuevas
df["anio"] = df["nombre_columna"].dt.year
df["mes"] = df["nombre_columna"].dt.month
```

### Lo que datetime habilita (accesor `.dt`)

```python
df["year"] = df["lastUpdated"].dt.year      # columna nueva con el año
df["month"] = df["lastUpdated"].dt.month    # columna nueva con el mes

# Orden cronológico real + vista de columnas seleccionadas
df.sort_values("lastUpdated").head(5)[["country_id", "lastUpdated", "year", "month", "value"]]
```

---

## 20. Regla general: conversiones seguras con `errors="coerce"`

**Cuándo usarlo:** SIEMPRE que conviertas tipos en datos reales. Los datasets nunca son 100% limpios.

| Conversión | Código | Inválidos se vuelven |
|---|---|---|
| A número | `pd.to_numeric(df[col], errors="coerce")` | `NaN` |
| A fecha | `pd.to_datetime(df[col], errors="coerce")` | `NaT` |

**Por qué:** en vez de romper el notebook, los valores inválidos quedan marcados como nulos — auditables. Los KPIs se calculan solo sobre datos válidos y los casos dudosos quedan explícitos para imputar o filtrar después.

**Cierre del flujo:** valida siempre con `df.dtypes` — cada columna con el tipo que le corresponde (numéricas en `float64`/`int64`, temporales en `datetime64[ns]`).

---

[← Valores nulos y duplicados](02-nulos-y-duplicados.md) · [Índice](../README.md) · [Filtrado y estadística descriptiva →](04-filtrado-y-estadistica.md)
