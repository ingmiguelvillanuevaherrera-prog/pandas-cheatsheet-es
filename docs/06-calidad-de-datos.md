# Diagnóstico de calidad e imputación

[← Índice de la guía](../README.md)

---

## 30. Revisión inicial: roles, faltantes e inválidos (diagnóstico de calidad)

> Antes de limpiar o automatizar, entiende qué contiene el dataset: qué columnas hay, qué representan y qué tan confiable es cada una. Es el punto de partida de todo sprint.

**Cuándo usarlo:** la primera vez que abres un dataset, antes de aplicar reglas de limpieza. Responde tres preguntas en orden: (1) ¿qué **rol** y **tipo** tiene cada columna? (2) ¿dónde **faltan** datos? (3) ¿qué valores presentes son **inválidos**?

---

### 30.1 Clasificar columnas por rol y tipo

**Idea central:** el **tipo** (lo que detecta Python: `int`, `float`, `object`, `datetime`) **NO es lo mismo** que el **rol** (lo que significa la columna en el negocio). Un `customer_id` es `int` pero su rol es **ID**, no una variable numérica.

```python
df.info()    # tipos, no-nulos y memoria — tu resumen inicial (sección 2)
df.head()    # ver el contenido real de las primeras filas
```

| Rol | Qué es | Ejemplos | ¿Se analiza estadísticamente? |
|---|---|---|---|
| ID / Llave | Identifica cada registro o entidad | `order_id`, `customer_id` | **No** — aunque sea `int`, no se promedia |
| Numérica | Cantidad medible, permite operaciones | `customer_age`, `quantity`, `order_value` | Sí |
| Categórica | Etiqueta o grupo (normalmente `object`) | `product_category`, `city`, `state` | Se cuenta/agrupa, no se promedia |
| Fecha | Marca temporal — convertir a `datetime` (sección 19) | `order_date` | Habilita estacionalidad y tendencias |

⚠️ **Nunca confíes ciegamente en `.info()`** — úsalo como guía, pero revísalo:

- Un **ID** aparece como `int` y parece numérico, pero **no se analiza** como tal.
- Símbolos como `"?"` o `"%"` hacen que una columna numérica salga como `object` (conecta con la sección 17–18: convertir texto a número).
- Los **sentinels** (ej. `999`) **no cambian el tipo**, pero corrompen el análisis porque no son datos reales (ver 30.3).

**Por qué importa para el negocio:** una clasificación incorrecta lleva a conclusiones equivocadas. Si `customer_age` no se lee como numérica → no hay análisis de edades. Si `product_category` no se trata como categoría → se pierden patrones por producto. Si `order_date` no es fecha → no se ve estacionalidad.

#### 🧩 Cardinalidad — la huella de cada columna

**Cuándo usarlo:** junto con `.info()`, para reconocer el rol de cada columna por su número de valores únicos. Extiende el uso de `.nunique()` de la sección 15 (antes por columna; aquí sobre **todo** el DataFrame).

**🔁 Sintaxis genérica:**

```python
# Valores únicos por columna, ordenados de menor a mayor
df.nunique().sort_values()
```

| Cardinalidad | Sugiere | Ejemplos | Problema potencial |
|---|---|---|---|
| Muy alta (≈ nº de filas) | ID / clave única | `order_id` (5008), `customer_id` (1829) | Pocos únicos → IDs duplicados o pérdida de datos |
| Alta | Métrica continua o fecha | `order_value` (3889), `price` (1677), `order_date` (381) | Muy baja → precios redondeados o periodo mal cargado |
| Media | Numérica discreta | `customer_age` (64), `quantity` (249) | Muy pocos → redondeo o errores |
| Baja | Categórica | `payment_method` (4), `product_category` (8), `state` (9), `city` (10) | Menos de lo esperado → datos incompletos |

⚠️ **Error #7 de tu lista:** `.nunique()` **cuenta valores únicos**, no tiene nada que ver con `.dtypes` (tipos). No los confundas.

📌 **Conecta con la sección 37.** Una vez clasificada la columna por rol, la 37.3 te dice qué medidas aplicarle: numérica → `.mean()` / `.std()` / percentiles (sección 22); categórica → `.value_counts()` / `normalize=True` / `.mode()`. Y ⚠️ la 37.5 documenta por qué `unique` por sí solo engaña: cuenta sentinels y valores imputados (`"?"`, `"unknown"`) como si fueran categorías reales.

---

### 30.2 Identificar valores faltantes (%)

**Cuándo usarlo:** tras clasificar roles, para medir la **severidad** de los huecos (no basta con saber que existen).

**Idiom nuevo — `.isna().mean()`:** `.isna()` devuelve `True`/`False` por celda; `.mean()` sobre booleanos calcula la **proporción de `True`** (porque `True`=1 y `False`=0). Es una forma más limpia de tu fórmula de la sección 6 (`df.isna().sum() / len(df) * 100`) — mismo resultado, expresado como proporción de 0 a 1.

**🔁 Sintaxis genérica:**

```python
# Proporción de nulos por columna (0 a 1), mayor a menor
df.isna().mean().sort_values(ascending=False)

# En porcentaje, redondeado
(df.isna().mean() * 100).round(2).sort_values(ascending=False)
```

**Umbrales de decisión (sección 6):** `< 5%` sin problema · `5–30%` depende de la importancia · `≥ 50%` columna poco confiable, márcala.

**Interpretación tipo analista (dataset de ejemplo):** nivel general **bajo** de faltantes.

- `customer_age` (2.99%) → el más ausente; afecta análisis demográfico y segmentación.
- `city` / `state` (1.99%) → afecta reportes geográficos por región.
- `order_date` (0.16%) → mínima, pero relevante para series temporales.
- El resto en 0% → ventas, órdenes, productos y método de pago están completos.

---

Regla mental: *antes de analizar, cada valor debe ser válido, posible y consistente con la lógica del negocio.*

### 30.3 Identificar valores inválidos (presentes pero falsos)

**Distinción clave:** un **vacío** (celda sin valor, `NaN`) **no es lo mismo** que un **inválido** (hay un valor, pero no representa información real: `"?"`, `999`, `quantity <= 0`). ⚠️ **`.isna()` NO detecta los inválidos** — hay que buscarlos columna por columna con **lógica de negocio**.

**Categórica → categorías inesperadas o corruptas** (usa `.isin()` de la sección 21):

```python
df["product_category"].value_counts()        # ¿aparece "?" u otra categoría rara?
df[df["product_category"].isin(["?"])]        # aislar los registros sospechosos
```

**Numérica → valores imposibles:**

```python
df["quantity"].le(0).sum()    # cuántas cantidades son <= 0 (imposible en e-commerce)
```

`.le(0)` = *less or equal* (`<= 0`); es el equivalente en **método** de tu `(df["quantity"] <= 0).sum()` de la sección 14.

**Numérica → rangos improbables (delata sentinels):**

```python
df[["customer_age", "quantity"]].describe()   # min/max absurdos saltan a la vista (ej. -999)
df["customer_age"].isin([-999]).sum()         # contar el valor centinela exacto
```

**🔁 Sintaxis genérica:**

```python
# Categórica: ver categorías y aislar la inválida
df["nombre_columna"].value_counts()
df[df["nombre_columna"].isin(["valor_invalido"])]

# Numérica: contar valores imposibles con método comparativo
df["nombre_columna"].le(valor_limite).sum()   # <= ; usa .lt/.ge/.gt según la regla

# Rango: describe delata mínimos/máximos absurdos, luego cuenta el centinela
df["nombre_columna"].describe()
df["nombre_columna"].isin([valor_centinela]).sum()
```

| Método | Equivale a | Pregunta que responde |
|---|---|---|
| `.lt(x)` | `< x` | ¿Menor que el límite? |
| `.le(x)` | `<= x` | ¿Menor o igual? (ej. cantidades no positivas) |
| `.gt(x)` | `> x` | ¿Mayor que el límite? (ej. edades \> 100) |
| `.ge(x)` | `>= x` | ¿Mayor o igual? |
| `.eq(x)` | `== x` | ¿Igual a un valor exacto? (ej. el centinela) |

#### 💡 Sentinels (valores centinela) — inválidos disfrazados

Un **sentinel** es un valor especial que **marca** algo (un faltante, un estado especial), **no** un dato real. Antes de que existiera `NaN`, se usaban `999`, `-1`, `-999`, `"UNKNOWN"`, `"?"` para indicar "vacío" o "desconocido".

**Por qué son peligrosos:**

- Pasan el filtro de `.isna()` (no son nulos) y **no cambian el `dtype`** → son invisibles a tu diagnóstico normal.
- Envenenan las estadísticas: un `-999` en `customer_age` **hunde la media**. Conecta con la sección 22 (media vs mediana): un sentinel actúa como un outlier artificial que separa media y mediana.

**Cómo detectarlos:** `min`/`max` absurdos en `.describe()`, un valor con frecuencia sospechosamente alta en `value_counts()`, o números "redondos" tipo `999` / `-999` / `0`.

**Qué hacer:** reemplazarlos por `NaN` y tratarlos después con tu flujo de nulos (secciones 6 y 9: imputar mediana o filtrar). Nunca los dejes como número real.

#### Vacío vs Inválido (resumen)

| Tipo | Qué es | Cómo se detecta |
|---|---|---|
| Vacío | Celda sin valor (`NaN` / `NaT`) | `.isna().sum()` / `.isna().mean()` (sección 6) |
| Inválido | Valor presente pero falso (`"?"`, `999`, `<= 0`) | `value_counts()`, `.describe()`, condiciones + lógica de negocio |

---

### Flujo completo del diagnóstico inicial (orden importa)

1. `df.info()` + `df.head()` → tipos y contenido real.
2. `df.nunique().sort_values()` → reconocer roles por cardinalidad.
3. `df.isna().mean().sort_values(ascending=False)` → % de faltantes y su severidad.
4. Por columna clave: `value_counts()` / `.describe()` / condiciones → inválidos y sentinels.
5. Documenta qué columnas son **confiables** antes de tocar nada.

📘 Enlaza con la **regla #4 de tus revisores**: cada cifra del diagnóstico (% de faltantes, conteo de inválidos) sale de una salida de código — nada "a ojo".

### Términos clave

Revisión inicial · Variable numérica · Variable categórica · ID/Llave · Vacío · Inválido · Cardinalidad · **Sentinel**.

---

## 31. Manejar valores ausentes e inválidos (MCAR/MAR/MNAR + imputación)

> No todos los faltantes significan lo mismo, ni se limpian igual. Y los inválidos NO se tratan como los ausentes.

**Cuándo usarlo:** después del diagnóstico inicial (sección 30), cuando ya sabes **dónde** están los huecos e inválidos y toca decidir **qué hacer** con ellos: eliminar, imputar o marcar con una bandera (flag).

---

### 31.1 Los tres tipos de missingness (¿por qué falta?)

Antes de tocar un dato, entiende **por qué** falta. No es lo mismo una celda vacía por un fallo técnico que una vacía solo en cierto tipo de cliente.

| Tipo | Significado | La ausencia depende de… | Qué hacer |
|---|---|---|---|
| **MCAR** | Missing Completely At Random (faltante completamente al azar) | Nada — ruido puro | Si el % es bajo: eliminar o imputar simple, sin sesgo importante |
| **MAR** | Missing At Random (faltante al azar) | Otra variable **observada** (ej. ciudad, canal, método de pago) | Imputar por grupo con esa variable; documentar el sesgo |
| **MNAR** | Missing Not At Random (faltante no al azar) | El **propio valor** faltante o un proceso no observable | Flag de missing, tratar "la ausencia como señal", ser muy transparente. **No imputar a ciegas** |

**Ejemplos para fijar la diferencia:**

- **MCAR** — un fallo técnico borra celdas al azar durante la carga. Ninguna fila tiene más probabilidad que otra de quedar vacía.
- **MAR** — los campos de dirección faltan más cuando el pedido viene de cierto **canal de venta** o **método de pago**. Dentro de ese canal, todas las ciudades son igual de probables: el hueco lo explica una columna que **sí tienes**.
- **MNAR** — los clientes con **ingresos más altos** son justo los que no completan el campo "ingreso", porque no quieren revelarlo. Aquí la ausencia **es** parte del fenómeno que quieres estudiar: si imputas la media, borras exactamente la señal que buscabas.

📘 **Regla #3 de tus revisores** (hipótesis ≠ causa): en MNAR sobre todo, se sugiere y se documenta ("podría deberse a…"), no se afirma la causa sin evidencia.

---

### 31.2 Diagnosticar el tipo — caso `customer_age`

Es la columna con más vacíos (\~3%). Para saber su tipo, buscamos patrón en **dos pasos**.

#### Paso 1 — ¿Los faltantes dependen de un grupo? (MAR)

Revisa si los vacíos se concentran en ciertas ciudades o categorías:

```python
df["customer_age"].isna().groupby(df["city"]).mean().sort_values(ascending=False)
df["customer_age"].isna().groupby(df["product_category"]).mean()
```

**Cómo leerlo:** `.isna()` da True/False; `.groupby(df["grupo"]).mean()` calcula la **proporción de faltantes por grupo** (media de booleanos = proporción, mismo truco de la sección 30.2). Si una ciudad/categoría concentra muchos más vacíos → la ausencia depende de ese grupo → **MAR**. En el dataset de ejemplo **no** hay patrón fuerte → no es MAR.

**🔁 Sintaxis genérica:**

```python
# Proporción de faltantes de una columna, desglosada por grupo
df["columna_con_nulos"].isna().groupby(df["columna_grupo"]).mean().sort_values(ascending=False)
```

#### Paso 2 — ¿El propio valor es especial? (MNAR)

Crea una **bandera (flag)** y compara el comportamiento de compra de quien tiene edad vs quien no:

```python
# Bandera binaria: 1 = edad faltante, 0 = presente
df["edad_vacia"] = df["customer_age"].isna().astype(int)

# ¿Compran distinto los que no tienen edad?
df.groupby("edad_vacia")["order_value"].describe()
```

`.astype(int)` convierte True/False en 1/0. Si el grupo sin edad tiene un `order_value` muy distinto → la ausencia se liga al propio valor → **MNAR**. En el dataset de ejemplo los dos grupos se comportan **parecido** → no es MNAR.

**Conclusión:** descartados MAR y MNAR → `customer_age` es **MCAR** (faltante completamente al azar).

**🔁 Sintaxis genérica (flag + comparación):**

```python
df["flag_vacio"] = df["nombre_columna"].isna().astype(int)
df.groupby("flag_vacio")["columna_comportamiento"].describe()
```

| Pregunta | Chequeo | Si sí… |
|---|---|---|
| ¿Los faltantes se concentran en un grupo? | `isna().groupby(df["grupo"]).mean()` | MAR |
| ¿Los faltantes se comportan distinto? | flag + `groupby(flag)["metrica"].describe()` | MNAR |
| Ni patrón de grupo ni comportamiento especial | los dos anteriores salen "sin patrón" | MCAR |

---

### 31.3 Imputar `customer_age` (MCAR → mediana)

Si es **MCAR** y solo \~3%, la mejor práctica es un método simple y sin sesgo: la **mediana** (retoma la sección 9 `fillna` y sección 22 media vs mediana).

```python
# 1) Calcular la mediana
median_age = df["customer_age"].median()

# 2) Rellenar y GUARDAR
df["customer_age"] = df["customer_age"].fillna(median_age)

# 3) Validar — debe dar 0
df["customer_age"].isna().sum()
```

**Por qué mediana y no media:** la mediana es robusta a outliers (edades extremas no la desplazan); mantiene la distribución. ⚠️ **Error #5 de tu lista:** si la columna fuera `object`, conviértela con `pd.to_numeric(..., errors="coerce")` ANTES de la mediana.

**Otras opciones (no necesarias aquí):** imputar por grupo (si fuera MAR) · eliminar filas (con 3% se puede, pero perderías datos si puedes imputar bien).

💡 Buena práctica: **conserva la bandera** `edad_vacia` — deja auditable qué filas eran originalmente vacías ("ausencia como señal").

---

### 31.4 Limpiar `quantity <= 0` (INVÁLIDO, no ausente)

❗ **Los inválidos NO son MCAR/MAR/MNAR** — no son ausencias, son **errores**. `quantity <= 0` no tiene sentido en un pedido (retoma sección 14: valores imposibles). Casi 3.000 registros aquí.

**¿Por qué existen? (hipótesis, no certeza)** Lo más probable es un fallo de captura o de cálculo al construir el dataset: celdas de `quantity` que nunca se llenaron y quedaron en `0`. 📘 **Regla #3** — esto se **sugiere y se documenta**, no se afirma. Lo que sí puedes afirmar es el **conteo**, porque sale de una salida de código.

Flujo: **marcar como NaN → validar una fórmula de reconstrucción → imputar**.

#### Paso 1 — Marcar los inválidos como `NaN`

`fillna()` solo actúa sobre `NaN` reales, así que primero conviertes el inválido en `NaN`:

```python
import numpy as np

# Donde quantity <= 0, poner NaN
df.loc[df["quantity"] <= 0, "quantity"] = np.nan
```

**Nuevo — `df.loc[condición, "columna"] = valor`:** asignación condicional. `.loc[filtro_de_filas, columna]` selecciona las filas que cumplen la condición **en esa columna** y les asigna el valor. Es la forma correcta de "editar solo las filas que cumplen X".

#### Paso 2 — Validar la fórmula ANTES de imputar

Sabemos que `quantity = order_value / price`. Verifica que el dataset ya sigue esa lógica en las filas válidas:

```python
df["calculated_quantity"] = df["order_value"] / df["price"]
df[["order_value", "price", "quantity", "calculated_quantity"]].head()
```

Si `calculated_quantity` coincide con `quantity` en las filas buenas → el dataset se construyó con esa relación → es seguro reconstruir sin inventar datos. 📘 Esto es la **regla #4 de tus revisores**: no imputas a ciegas, validas la lógica con una salida de código.

#### Paso 3 — Imputar solo los `NaN` con la fórmula validada

```python
df["quantity"] = df["quantity"].fillna(df["order_value"] / df["price"])
# .round() si la cantidad debe ser entera:
# df["quantity"] = df["quantity"].fillna((df["order_value"] / df["price"]).round())
```

`fillna()` con una Serie rellena **cada** `NaN` con el valor correspondiente de `order_value / price`, dejando intactas las filas que ya tenían un valor válido.

**🔁 Sintaxis genérica:**

```python
# Marcar inválidos como NaN
df.loc[df["nombre_columna"] <= valor_limite, "nombre_columna"] = np.nan

# Validar una fórmula de reconstrucción
df["col_calculada"] = df["col_a"] / df["col_b"]
df[["col_a", "col_b", "nombre_columna", "col_calculada"]].head()

# Imputar SOLO los NaN con la fórmula validada
df["nombre_columna"] = df["nombre_columna"].fillna(df["col_a"] / df["col_b"])
```

---

### 31.5 Evaluación pre–post — ¿mi limpieza cambió las conclusiones?

**Cuándo usarlo:** justo después de imputar o filtrar, antes de analizar. Es el paso que convierte tu limpieza en **evidencia**: demuestra en números cuánto movió la aguja.

**Idea central:** limpiar no es cosmético. Si la media de `quantity` cambia después de reparar \~3.000 registros inválidos, entonces **cualquier conclusión sacada antes de limpiar estaba equivocada**. Eso es un hallazgo de negocio, no un detalle técnico.

⚠️ **Requisito:** guarda una copia ANTES de tocar nada (**sección 5**). Sin ella no hay comparación posible.

**🔁 Sintaxis genérica:**

```python
# ANTES de limpiar (sección 5)
df_original = df.copy()

# DESPUÉS de limpiar — comparar la misma métrica
print("Antes: ", df_original["nombre_columna"].mean())
print("Después:", df["nombre_columna"].mean())

# Comparación completa de un jalón
comparacion = pd.DataFrame({
    "antes":   df_original["nombre_columna"].describe(),
    "despues": df["nombre_columna"].describe()
})
comparacion["cambio_pct"] = ((comparacion["despues"] - comparacion["antes"])
                             / comparacion["antes"] * 100).round(2)
comparacion
```

**Qué comparar (mínimo):**

| Métrica | Qué te dice si cambió |
|---|---|
| `count` | Cuántas filas ganaste (imputación) o perdiste (filtrado) |
| `mean` | Si había sentinels u outliers envenenando el promedio |
| `median` | Si el cambio es **estructural** — la mediana es robusta, así que si ELLA se movió, el problema era grande |
| `std` | Si la imputación achicó artificialmente la variabilidad |
| `min` / `max` | Si desaparecieron los valores imposibles |

⚠️ **Efecto secundario de imputar con la mediana:** rellenar muchos huecos con un mismo valor **reduce la desviación estándar** y concentra la distribución artificialmente. Con \~3% (caso `customer_age`) es despreciable; con 30% ya distorsiona. Repórtalo siempre.

💼 **Cómo se escribe esto en un portafolio:** *"Reparar los N registros con `quantity <= 0` movió la cantidad promedio por pedido de X a Y (+Z%) — el análisis previo subestimaba el volumen real de unidades vendidas."* Un revisor lee eso como criterio, no como sintaxis.

📘 Enlaza con la **regla #4 de tus revisores**: cada cifra sale de una salida de código — incluida la del "antes".

---

### Ausente vs Inválido — cómo se tratan (resumen)

| Caso | Qué es | Estrategia |
|---|---|---|
| Ausente (`NaN`) | Falta el dato | Clasifica MCAR/MAR/MNAR → eliminar / imputar / flag según el tipo |
| Inválido (`<= 0`, `999`, `"?"`) | Hay valor, pero es falso | 1º convertir a `NaN` (`.loc`), 2º imputar (fórmula/mediana) o eliminar |

### Flujo completo (orden importa)

1. Diagnostica el tipo de faltante: MAR con `isna().groupby().mean()`; MNAR con flag + `groupby().describe()`.
2. Elige estrategia: **MCAR** bajo → mediana; **MAR** → imputar por grupo; **MNAR** → flag + documentar.
3. Para **inválidos**: márcalos `NaN` con `.loc`, valida la lógica de reconstrucción, luego imputa.
4. Valida siempre: `df["col"].isna().sum()` → **0**.
5. **Compara pre–post** (31.5): `mean`, `median`, `std`, `count` antes vs después — si la conclusión cambió, eso es un hallazgo.
6. Documenta cada decisión (reglas #2 y #3 de tus revisores).

### ⚠️ Errores a vigilar (de tu lista)

- **Error #2:** guarda con `df["col"] = ...` al imputar, o el cambio se pierde.
- **Error #5:** mediana sobre columna `object` → convierte con `pd.to_numeric` primero.
- `fillna()` solo actúa sobre `NaN` reales → marca antes los inválidos como `NaN`.
- No imputar a ciegas un MNAR ni afirmar causa sin evidencia (regla #3).

### Términos clave

MCAR · MAR · MNAR · Imputación (`fillna`) · Flag de missingness (bandera binaria con `.astype(int)`) · Asignación condicional (`df.loc[cond, col] = ...`) · Evaluación pre–post (comparar métricas antes/después de limpiar).

---

[← Combinar datasets y exportar](05-merge-y-exportacion.md) · [Índice](../README.md) · [Python aplicado a la limpieza →](07-python-para-limpieza.md)
