# Valores nulos y duplicados

[← Índice de la guía](../README.md)

---

## 6. Diagnóstico de nulos

```python
df.isna()                  # True/False por celda
df.isna().sum()            # nulos POR COLUMNA
df.isna().sum().sum()      # TOTAL global ("la suma de la suma")
df.isna().sum() / len(df) * 100    # porcentaje por columna o proporción

df.isna().any()            # ¿la columna tiene al menos 1 nulo? (True/False)
df.columns[df.isna().any()]        # nombres de columnas con nulos
```

### Umbrales de decisión

| % de nulos | Decisión |
|---|---|
| \< 5% | Sin problema: rellena o elimina filas |
| 5–30% | Depende de la importancia de la columna |
| ≥ 50% | Columna poco confiable — no imputes, márcala |

### Nulos disfrazados (`"NULL"`, `"N/A"`, `""`, `" "`)

```python
df["unit"].value_counts(dropna=False)   # dropna=False incluye los NaN en el conteo
```

Si ves tokens raros, `isna()` está **subestimando** los huecos.

**🔁 Sintaxis genérica:**

```python
# Cuenta cada valor de 'nombre_columna', incluyendo los NaN
df["nombre_columna"].value_counts(dropna=False)
```

---

## 7. Filtrar filas con condición

**Estructura: `df[ condición ]` — el DataFrame va AFUERA y ADENTRO.**

```python
# Esqueleto
df_clean[ df_clean["columna"].isna() ]

# Filtrar + seleccionar columna + método
df_clean[df_clean["value"].isna()]["country_name"].unique()        # valores únicos
df_clean[df_clean["value"].isna()]["Entity"].value_counts()        # conteo por valor
```

**Por capas:**

```python
df_clean[   df_clean["col"].isna()   ]["otra_col"].metodo()
↑ DF afuera ↑ condición adentro       ↑ columna    ↑ .unique() / .value_counts() / ...
```

❌ Error común: `df["col"].isna()["Entity"]` — una serie de True/False no tiene columnas. La condición va DENTRO de los corchetes del DataFrame.

---

## 8. Opción A — Eliminar nulos: `.dropna()`

```python
df_clean = df.dropna(subset=["value"]).reset_index(drop=True)
#                    ↑ solo filas donde ESTA columna es nula
#                                       ↑ reinicia índices sin huecos
```

Úsalo si los nulos son **pocos y dispersos** (no concentrados por país/grupo).

**🔁 Sintaxis genérica:**

```python
# Elimina filas donde 'nombre_columna' es nula y renumera el índice
df = df.dropna(subset=["nombre_columna"]).reset_index(drop=True)
```

---

## 9. Opción B — Imputar: `.fillna()`

**Flujo completo (orden importa):**

```python
# 0) Si la columna es object (texto), conviértela PRIMERO
df_clean[col] = pd.to_numeric(df_clean[col], errors="coerce")
#   errors="coerce" → lo que no se puede convertir se vuelve NaN (lo atrapa el fillna)

# 1) Calcula la mediana
mediana = df_clean[col].median()

# 2) Rellena y GUARDA (asignación de vuelta a la columna)
df_clean[col] = df_clean[col].fillna(mediana)

# 3) Valida — debe dar 0
df_clean[col].isna().sum()
```

**¿Por qué mediana y no promedio?** La mediana es robusta a outliers: un pico extremo no la desplaza.

⚠️ Usa **solo una** de las dos opciones (dropna O fillna) sobre la misma columna.

---

## 10. Validación final

```python
df_clean.isna().sum()     # sin nulos en la métrica clave
df_clean.dtypes           # tipos correctos (métrica debe ser int64/float64, no object)
```

---

## 11. Diccionario de datos — antes de programar

Tres decisiones que salen del diccionario, no de tu intuición:

1. ¿Qué columnas **eliminar**? (irrelevantes)
2. ¿Qué columnas **convertir**? (fechas, números como texto)
3. ¿Qué columnas tienen **reglas de validación**? (ej. `country_id` = ISO de 2 letras)

---

## 12. Diccionarios de Python (la estructura, no el de datos)

```python
paises = {
    "MX": "México",      # "clave": valor
    "PE": "Perú",
}
```

Pares `clave: valor`, separados por comas, entre llaves `{}`. Se usan en `.rename(columns={...})`.

---

### ⚠️ Errores que ya cometiste (para no repetir)

1. `df.copy` sin paréntesis → no ejecuta nada. Métodos siempre con `()`.
2. Olvidar `df = ...` al renombrar/rellenar → el cambio se pierde.
3. Condición fuera de los corchetes del DF → `df[condición]`, no `df["col"].isna()["otra"]`.
4. Recargar el CSV después de limpiar → adiós limpieza.
5. Calcular mediana sobre columna `object` → convierte con `pd.to_numeric` primero.
6. Mezclar `df` y `df_clean` en el mismo flujo → después de `df_clean = df.copy()`, el original `df` NO debe aparecer en ninguna línea más. Si limpias uno y validas el otro, tu validación miente. Truco: Ctrl+F buscando `df[` o `df.` sueltos.
7. Confundir `.dtype` / `.dtypes` / `.nunique()` → `df["col"].dtype` = tipo de UNA columna; `df.dtypes` = tipos de TODAS; `.nunique()` = cuenta valores únicos (nada que ver con tipos).

---

## 13. Eliminar duplicados

**Cuándo usarlo:** después de resolver nulos, antes de calcular promedios/KPIs. Si el mismo registro aparece dos veces, tus promedios se inflan y los comparativos engañan.

### Detectar duplicados exactos (fila completa idéntica)

```python
df.duplicated().sum()        # cuántas filas son copia exacta (cuenta desde la 2ª aparición)

df[df.duplicated(keep=False)].head(10)   # ver ejemplos — keep=False muestra TODAS las apariciones
```

**Cuándo:** primer chequeo siempre. Si da \> 0, hay doble envío o reproceso de archivos.

### Detectar duplicados por llave de negocio

Los duplicados importantes casi nunca son la fila completa — son registros que comparten la misma clave (ej. misma ciudad + mismo instante).

```python
# 1) Convertir la columna temporal a datetime PRIMERO (evita falsos positivos por formato)
df["lastUpdated"] = pd.to_datetime(df["lastUpdated"], errors="coerce")

# 2) Definir la llave
keys = ["city", "lastUpdated"]

# 3) Contar duplicados por llave
df.duplicated(subset=keys).sum()

# 4) Ver ejemplos ordenados por llave
df[df.duplicated(subset=keys, keep=False)].sort_values(keys).head(20)
```

**Cuándo:** cuando la combinación de columnas debe ser única (una medición por ciudad por instante, una venta por ticket, etc.).

### Eliminar duplicados

```python
keys = ["city", "lastUpdated"]
df = df.drop_duplicates(subset=keys, keep="first").reset_index(drop=True)

# Validación — debe dar 0
df.duplicated(subset=keys).sum()
```

- `subset=keys` → compara solo por la llave, no la fila completa.
- `keep="first"` → conserva la primera aparición, descarta el resto. Suficiente cuando el archivo llega en orden de ingestión.
- `reset_index(drop=True)` → renumera los renglones e ignora el índice viejo.
- **Guarda con `df = ...`** o el cambio se pierde.

### Parámetros de `duplicated()` / `drop_duplicates()`

| Parámetro | Qué hace | Cuándo usarlo |
|---|---|---|
| (sin argumentos) | Compara la fila COMPLETA | Chequeo inicial de copias exactas |
| `subset=["col1", "col2"]` | Compara solo esas columnas | Duplicados por llave de negocio |
| `keep="first"` | Conserva la primera aparición | Al eliminar (default) |
| `keep=False` | Marca TODAS las apariciones | Al inspeccionar antes de borrar |

### Flujo completo (orden importa)

1. `pd.to_datetime()` sobre la columna temporal de la llave.
2. Contar: `df.duplicated(subset=keys).sum()`.
3. Inspeccionar: `df[df.duplicated(subset=keys, keep=False)].sort_values(keys)`.
4. Eliminar: `drop_duplicates(subset=keys, keep="first").reset_index(drop=True)`.
5. Validar: el conteo del paso 2 debe dar **0**.

### 🔁 Sintaxis genérica (copiar y adaptar)

```python
# Cuenta filas duplicadas exactas (fila completa)
df.duplicated().sum()

# Cuenta llaves repetidas según columnas clave
keys = ["columna_1", "columna_2"]
df.duplicated(subset=keys).sum()

# Elimina duplicados por llave, conserva la primera aparición y renumera
df = df.drop_duplicates(subset=keys, keep="first").reset_index(drop=True)
```

---

## 14. Filtrar filas irrelevantes (valores imposibles)

**Cuándo usarlo:** cuando hay valores sin sentido físico o de negocio (ej. PM2.5 ≤ 0, edades negativas, precios en 0). Sin duplicados también puede haber ruido.

### Flujo: diagnosticar → filtrar → reindexar

```python
# 1) DIAGNÓSTICO — cuántas filas violan la regla y qué % representan
(df["value"] <= 0).sum()                  # conteo de valores no físicos
df.shape                                  # tamaño ANTES del filtro
(df["value"] <= 0).sum() / len(df) * 100  # impacto en %

# 2) FILTRO — conservar solo lo válido (nota: la condición se INVIERTE)
df = df[df["value"] > 0]
df.shape                                  # tamaño DESPUÉS — cuantifica la reducción

# 3) REINDEXAR — renumerar sin huecos
df = df.reset_index(drop=True)
df.head(10)                               # confirmar índice continuo (0, 1, 2, ...)
```

**Detalles clave:**

- La condición del diagnóstico (`<= 0`) y la del filtro (`> 0`) son **opuestas**: una cuenta lo malo, la otra conserva lo bueno.
- Siempre diagnostica ANTES de filtrar: si el % es grande, documéntalo — afecta comparabilidad.
- El paréntesis en `(df["value"] <= 0).sum()` es necesario para aplicar `.sum()` a la condición completa.
- Guarda con `df = ...` o el filtro se pierde.

### 🔁 Sintaxis genérica (copiar y adaptar)

```python
# Cuenta filas que violan la regla de plausibilidad
(df["nombre_columna"] <= valor_limite).sum()

# Conserva solo filas válidas y renumera el índice
df = df[df["nombre_columna"] > valor_limite].reset_index(drop=True)
```

---

## 15. Auditar y validar la limpieza

**Cuándo usarlo:** SIEMPRE al final, después de quitar nulos, duplicados y valores imposibles. Es tu evidencia de que el dataset quedó coherente.

### Los 4 chequeos

```python
# 1) Cobertura — ¿cuántas categorías únicas quedaron?
df["city"].nunique()

# 2) Distribución — ¿hay concentración o desbalance?
df["city"].value_counts()

# 3) Unicidad por llave — debe dar 0
df.duplicated(subset=["city", "lastUpdated"]).sum()

# 4) Tamaño antes vs. después (requiere haber guardado copia al inicio)
filas_antes = len(df_raw)
filas_despues = len(df)
print("Filas antes:", filas_antes, "Filas después:", filas_despues)
```

**Interpretación:** una reducción leve = solo se eliminó basura (duplicados/inválidos). Una reducción grande = documéntala, puede sesgar comparativos.

### 🔁 Sintaxis genérica (copiar y adaptar)

```python
df["nombre_columna"].nunique()                    # cuántos valores únicos hay
df["nombre_columna"].value_counts()               # conteo de cada valor
df.duplicated(subset=["col_1", "col_2"]).sum()    # llaves repetidas (debe dar 0)
```

### Auditoría: qué método para qué pregunta

| Pregunta | Método |
|---|---|
| ¿Cuántos valores distintos hay? | `.nunique()` |
| ¿Cuáles son esos valores? | `.unique()` |
| ¿Cuántas veces aparece cada uno? | `.value_counts()` |
| ¿Quedan llaves repetidas? | `.duplicated(subset=keys).sum()` → 0 |
| ¿Cuántas filas perdí? | `len(df_raw)` vs `len(df)` |

---

[← Carga e inspección inicial](01-carga-e-inspeccion.md) · [Índice](../README.md) · [Tipos de datos, texto y fechas →](03-tipos-texto-y-fechas.md)
