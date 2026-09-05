# Feature engineering

[← Índice de la guía](../README.md)

---

## 44. Feature engineering — crear una columna de segmento

**Cuándo usarlo:** cuando la pregunta de negocio necesita una variable que **no existe** en el dataset y hay que construirla a partir de reglas: segmentos de cliente, niveles de riesgo, indicadores, ratios.

📌 **La lógica (`if`, `elif`, `else`, operadores) está en la sección 43.** Aquí va cómo **aplicarla a un DataFrame entero**: `np.where()`, `.apply()`, y la distinción `&` vs `and`.

### 44.1 Qué es el feature engineering

Crear, transformar o combinar columnas para que el análisis revele algo que los datos crudos no muestran.

| Tipo | Ejemplo |
|---|---|
| Categorizar un rango numérico | edad → "joven" / "adulto" / "senior" |
| Agregar | sumar pedidos → gasto anual por cliente |
| Indicador binario (flag) | `edad_vacia` — ya lo hiciste en la sección **31.2** y **35.3** |
| Ratio entre columnas | ingresos / visitas |

📌 **Ya venías haciendo feature engineering sin llamarlo así:** las banderas de missingness de la 35.3 y la columna `_winsor` de la 42.2 son features creadas por ti.

### 44.2 Método 1 — `np.where()` anidado

`np.where(condición, valor_si, valor_si_no)` recorre la columna entera y asigna uno de dos valores. Es el equivalente vectorizado de un `if`/`else`.

```python
import numpy as np

df["customer_segment"] = np.where(df['customer_age'] >= 55, "Senior", "otro")
```

Para más de dos casos, se **anida**: el siguiente `np.where()` ocupa la posición del `valor_si_no`.

```python
df['customer_segment'] = np.where(
    (df['customer_age'] >= 55) & (df['order_value'] >= 10000), "Senior VIP",
        np.where((df['customer_age'] < 55) & (df['order_value'] >= 10000), "Junior VIP",
            np.where((df['customer_age'] >= 55) & (df['order_value'] >= 5000), "Sr. Medium Value",
                np.where((df['customer_age'] < 55) & (df['order_value'] >= 5000), "Jr. Medium Value",
                    np.where(df['order_value'] < 5000, "Low Value", "Error")
                )
            )
        )
)
```

⚠️ **Cada condición va entre paréntesis propios.** `df['a'] >= 55 & df['b'] >= 10000` **no** funciona: `&` tiene mayor precedencia que `>=`, así que Python intenta evaluar `55 & df['b']` primero y falla. Siempre `(cond1) & (cond2)`.

📘 **El truco del `"Error"` final es buena práctica y vale la pena copiarlo.** En vez de poner `"otro"` como último valor, pon `"Error"`: si todas tus reglas cubren todos los casos, esa categoría debe salir con **cero** registros. Es un **test que se valida solo**:

```python
df['customer_segment'].value_counts()   # ¿aparece "Error"? → tus reglas tienen un hueco
```

⚠️ **`np.where` NO es lo mismo que `.where()` de pandas:**

|   | Qué hace |
|---|---|
| `np.where(cond, a, b)` | Asigna en **ambos** casos: `a` si cumple, `b` si no |
| `serie.where(cond, b)` | **Conserva** el valor original si cumple, y lo reemplaza por `b` si no |

### 44.3 🎯 `&` / `|` vs `and` / `or` — la distinción que importa

|   | `and` / `or` | `&` / `\|` |
|---|---|---|
| Operan sobre | **Un** valor `True`/`False` | **Series** de pandas / arrays de NumPy, elemento a elemento |
| Se usan en | `if`, `while`, dentro de funciones | Filtrar DataFrames, `np.where()` |
| Paréntesis | No hacen falta | **Obligatorios** en cada condición |
| Ejemplo | `if spend >= 10000 and age >= 55:` | `df[(df["spend"] >= 10000) & (df["age"] >= 55)]` |

⚠️ **Usar `and` sobre una Serie da el mismo error de la sección 43.4:** `ValueError: The truth value of a Series is ambiguous`. Es la misma causa — `and` quiere un solo booleano y recibe 5,000.

💡 **Regla corta:** ¿estoy dentro de un `if`? → `and` / `or`. ¿Estoy operando sobre una columna? → `&` / `|` con paréntesis.

📌 Conecta con la **sección 21** (filtrado avanzado), donde ya usabas `&` sin que estuviera explicado el porqué.

### 44.4 Método 2 — función + `.apply(axis=1)`

El enfoque recomendado cuando hay varias condiciones.

```python
def classify_segment(row):
    age   = row['customer_age']
    spend = row['order_value']

    # Blindaje: los faltantes primero (ver 43.6)
    if pd.isna(age) or pd.isna(spend):
        return "Error en Datos"

    if spend >= 10000:
        if age >= 55:
            return "Senior VIP"
        else:
            return "Junior VIP"

    elif spend >= 5000:
        if age >= 55:
            return "Sr. Medium Value"
        else:
            return "Jr. Medium Value"

    else:
        return "Low Value"

df["customer_segment"] = df.apply(classify_segment, axis=1)
df["customer_segment"].value_counts()
```

| Pieza | Qué hace |
|---|---|
| `row` | La función recibe **una fila completa** cada vez. Es un valor escalar por columna, así que el `if` sí funciona (43.4) |
| `row['columna']` | Extrae el valor de esa columna **en esa fila** |
| **`axis=1`** | Recorre **fila por fila**. Sin esto (`axis=0`, el defecto) recibiría **columnas** enteras y todo se rompe |
| `return` | El valor devuelto se convierte en la celda de la nueva columna. **Con `print` no funcionaría** (32.5 y 43.5) |

💡 **Empieza con una función tonta.** Es un buen hábito: antes de escribir la lógica, prueba `return "otro"` y aplica. Si toda la columna sale con `"otro"`, la conexión función↔`apply` está bien y ya solo te falta la lógica. Separas el problema en dos.

### 44.5 `if` anidados vs `and` encadenado

Las dos versiones dan el **mismo resultado**:

```python
# Versión plana con and
if spend >= 10000 and age >= 55:
    return "Senior VIP"
elif spend >= 10000 and age < 55:
    return "Junior VIP"
elif spend >= 5000 and age >= 55:
    return "Sr. Medium Value"
elif spend >= 5000 and age < 55:
    return "Jr. Medium Value"
else:
    return "Low Value"
```

|   | `if` anidados | `and` encadenado |
|---|---|---|
| Repetición | Evalúa `spend >= 10000` **una vez** | Lo repite en cada rama |
| Legibilidad | Refleja el **árbol de decisión** del diagrama | Se lee como una lista de reglas |
| Al crecer | Añadir un tercer criterio = un nivel más de sangría | Añadir un criterio = multiplicar las ramas |

🎯 **No hay una correcta.** Si tu regla de negocio **es** un árbol (primero gasto, luego edad), anidar refleja esa jerarquía y se mantiene mejor. Si son reglas independientes, la versión plana se lee mejor. Con tres o más niveles de anidación, ninguna de las dos: ahí toca `pd.cut()` o una tabla de reglas.

### 44.6 Cuál método usar

| Criterio | `np.where()` anidado | Función + `.apply()` |
|---|---|---|
| Velocidad | **Vectorizado, mucho más rápido** | Fila por fila en Python puro, lento |
| Legibilidad | Se vuelve ilegible pasando de 2 condiciones | Se lee casi como la regla de negocio escrita |
| Mantener y reutilizar | Hay que reescribir el bloque entero | Editas la función; la reutilizas en otros DataFrames |
| Explicárselo a un gerente | Imposible | Directo |

💡 **Con 5,000 filas la diferencia de velocidad es imperceptible**, así que gana la legibilidad. La cosa cambia con millones de filas — ahí `.apply()` se nota y conviene la versión vectorizada, o `pd.cut()` para rangos.

📘 **En trabajo real, la regla de negocio va dentro de una función.** Es lo mismo que aprendiste en 34.3 y 35: la lógica encapsulada es reproducible, auditable y se prueba por separado.

### 44.7 ⚠️ Errores típicos al copiar este patrón

**1. Bug de indentación** — en el bloque con manejo de nulos:

```python
def classify_segment(row):
     age = row['customer_age']      # ← 5 espacios
    spend = row['order_value']      # ← 4 espacios
```

```text
IndentationError: unindent does not match any outer indentation level
```

Es el error #1 al empezar, y suele venir ya incluido en el código que copias. Dentro de un mismo bloque, todas las líneas llevan **exactamente la misma sangría**.

**2. El diagrama y el código no coinciden en las fronteras.**

| Caso | Dice el diagrama | Hace el código |
|---|---|---|
| Gasto = 10,000 exacto | "Gasto **mayor a** 10,000" → **no** es VIP | `spend >= 10000` → **sí** es VIP |
| Gasto = 9,999.50 | Banda "5,000 y 9,999" → **sin categoría** | `>= 5000` → Medium Value |
| Edad = 55 exacta | "mayor a 55" / "menor a 55" → **sin categoría** | `age >= 55` → Senior |

⚠️ **No es teórico:** `customer_age` son enteros de 18 a 80, así que hay clientes con **exactamente 55 años** —alrededor de 79 si la distribución es uniforme (36.7)— cayendo en un hueco que la especificación no define y que el código resuelve por su cuenta.

```python
(df["customer_age"] == 55).sum()   # ¿cuántos caen justo en la frontera?
```

📘 **Cuando la especificación dice "mayor a X" y el código dice `>= X`, alguien va a acabar reclamando.** Al escribir reglas de negocio, define siempre qué pasa **en** el umbral y deja el código y el documento diciendo lo mismo. Es el mismo cuidado de los valores frontera de la sección 43.6.

### 44.8 ⚠️ El problema de fondo: cada fila es una ORDEN, no un cliente

El enunciado mismo lo dice: *"cada fila representa una orden individual"*. Y a continuación se crea una columna llamada `customer_segment` a partir del `order_value` **de esa orden**, para responder *"¿cuántos clientes son Senior VIP?"*.

**Esa cuenta no da clientes, da órdenes.** Dos consecuencias concretas:

1. Un cliente con 5 pedidos se cuenta **5 veces** en el `value_counts()`.
2. Peor: ese mismo cliente puede salir **"Senior VIP" en un pedido grande y "Low Value" en uno chico**. La columna no describe al cliente; describe la orden.

Y el criterio de negocio era *"gasto total"*, no "valor de un pedido suelto". Un cliente con 10 pedidos de 3,000 gastó 30,000 y quedaría etiquetado como **Low Value** en las diez filas.

**Verifica el tamaño del problema:**

```python
print("Órdenes:", len(df))
print("Clientes únicos:", df["customer_id"].nunique())
print("Órdenes por cliente:", round(len(df) / df["customer_id"].nunique(), 2))
```

#### ✅ El flujo correcto: agregar primero, segmentar después

```python
# 1) Llevar los datos a nivel CLIENTE (sección 23: groupby + agg)
clientes = df.groupby("customer_id").agg(
    order_value=("order_value", "sum"),      # gasto TOTAL del cliente
    customer_age=("customer_age", "first")   # la edad no cambia entre pedidos
).reset_index()

# 2) Ahora sí, segmentar — la misma función, sin tocarla
clientes["customer_segment"] = clientes.apply(classify_segment, axis=1)

# 3) Y ahora la respuesta sí es en clientes
clientes["customer_segment"].value_counts()
```

🎯 **La regla general:** antes de segmentar, pregúntate **qué representa una fila**. Si la unidad de tu pregunta ("clientes") no coincide con la unidad de la tabla ("órdenes"), hay un `groupby` de por medio. Es la continuación exacta de lo que viste en 43.7 con el promedio.

💡 **Nota que sí conviene retener:** ordenar las condiciones de **mayor a menor prioridad** para que un VIP no quede capturado por una regla inferior (43.6).

### Términos clave

**Feature engineering** · `np.where(cond, a, b)` (vectorizado) vs `.where()` de pandas (conserva) · `np.where` anidado · **`&` / `|`** (elemento a elemento, con paréntesis) vs **`and` / `or`** (escalares) · Precedencia de operadores · `.apply(func, axis=1)` · `row['columna']` · **`axis=1`** (fila por fila) vs `axis=0` (columnas) · Categoría `"Error"` como test que se valida solo · `if` anidados vs condiciones planas · Vectorización vs iteración · **Unidad de análisis** (orden vs cliente) · Agregar antes de segmentar.

📌 **Conecta con:** sección 21 (`&` en filtros), 23 (`groupby` + `agg` para llevar a nivel cliente), 31.2 y 35.3 (flags, tus primeras features), 32.5 (`return` vs `print`), 34.3 (lógica encapsulada en función), 37 (`value_counts` para validar), 42.2 (la columna `_winsor` como feature), 43 (toda la lógica condicional).

---

[← Outliers: detección y tratamiento](09-outliers.md) · [Índice](../README.md)
