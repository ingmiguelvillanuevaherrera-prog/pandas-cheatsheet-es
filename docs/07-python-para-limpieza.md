# Python aplicado a la limpieza

[← Índice de la guía](../README.md)

---

## 32. Funciones para automatizar la limpieza (`def`)

> Una función es una máquina reutilizable: entra algo, se procesa, sale algo. Escribes la lógica **una vez** y la aplicas todas las veces que quieras.

**Cuándo usarlo:** cuando te descubres copiando y pegando la misma línea de limpieza cambiando solo el nombre de la columna. Ese copy-paste es la señal de que ahí va una función.

---

### 32.1 ¿Qué es una función?

🧺 **La analogía de la lavadora:** metes ropa sucia → la máquina hace su proceso → sale ropa limpia. La misma máquina sirve para ropa de color y para ropa blanca: lo que cambia es la **entrada**, no la máquina.

En Python se ve así:

```python
resultado = mi_funcion(entrada)
```

En limpieza de datos, esa "máquina" podría ser:

| Función | Qué recibe | Qué devuelve | Conecta con |
|---|---|---|---|
| `clean_age(valor_edad)` | una edad cruda | la edad validada | sección 31.3 |
| `normalize_category(texto_categoria)` | una categoría | la categoría estandarizada | sección 16 |
| `strip_and_lower(texto)` | un texto | sin espacios y en minúsculas | sección 16 |

Todas siguen el mismo patrón: **reciben un valor y regresan su versión limpia**.

---

### 32.2 El problema que resuelven: el código repetido

Empiezas con una línea:

```python
print("Hola Ana")
```

Y necesitas repetirla con otros nombres:

```python
print("Hola Ana")
print("Hola Carlos")
print("Hola Natalia")
```

**Observa qué pasa aquí:** todo el código es idéntico **excepto el nombre**. Esa parte que cambia es tu **entrada**; el resto es la máquina.

💡 **Regla práctica:** antes de escribir una función, mira tu código repetido y pregúntate *"¿qué es lo único que cambia?"*. Esa respuesta es tu **parámetro**.

---

### 32.3 Anatomía de una función

```python
def crear_saludo(nombre):
    print("Hola " + nombre)
```

| Parte | Qué es |
|---|---|
| `def` | Palabra clave que anuncia "voy a definir una función" |
| `crear_saludo` | El nombre que le das — así la llamarás después |
| `(nombre)` | El **parámetro**: la entrada que va a variar en cada uso |
| `:` | Cierra la línea de definición. **Siempre** va |
| Línea indentada | El **cuerpo**: lo que hace la función. La indentación NO es decorativa, define qué está dentro |

**Para usarla**, la mandas llamar por su nombre y le pasas la entrada:

```python
crear_saludo("Ana")
crear_saludo("Carlos")
crear_saludo("Natalia")
```

Salida:

```text
Hola Ana
Hola Carlos
Hola Natalia
```

⚠️ **Definir no es ejecutar.** El bloque `def` solo **guarda** la función; no pasa nada hasta que la llamas. Si corres solo el `def` y no ves salida, no está rota — nunca la invocaste.

**🔁 Sintaxis genérica (copiar y adaptar):**

```python
# 1) DEFINIR (esto solo la guarda)
def nombre_de_la_funcion(parametro):
    # lo que hace, siempre indentado
    print(parametro)

# 2) LLAMAR (esto sí la ejecuta)
nombre_de_la_funcion(valor)
```

---

### ⚠️ `print()` dentro NO es lo mismo que devolver

En el ejemplo de arriba el texto aparece en pantalla **porque hay un `print()` adentro**. Eso es distinto de una función que te **entrega** un valor para poder guardarlo.

Compara las dos formas:

```python
resultado = mi_funcion(entrada)   # ENTREGA algo → se puede guardar en una variable
crear_saludo("Ana")               # solo IMPRIME → no entrega nada guardable
```

Para limpieza de datos casi siempre quieres la primera: necesitas el valor limpio **de vuelta** para asignarlo a la columna (⚠️ **Error #2 de tu lista:** si no guardas, el cambio se pierde). La palabra que hace eso es `return`.

---

### 32.4 Los 4 componentes + convención de nombres

| # | Componente | Obligatorio | Qué hace |
|---|---|---|---|
| 1 | `def` | Sí | Le dice a Python "voy a definir una función" |
| 2 | Nombre de la función | Sí | `snake_case` (minúsculas + guiones bajos) — mismo estilo que ya usas para nombres de columnas (sección 28) |
| 3 | Parámetros | No | Entradas entre paréntesis; puede haber cero, una o varias |
| 4 | `return` | No | Lo que la función entrega como resultado |

```python
def saludar(nombre):
    return "Hola " + nombre
```

- `saludar` → nombre de la función
- `nombre` → parámetro de entrada
- `return "Hola " + nombre` → salida de la función

---

### 32.5 `return` — por qué es opcional (y qué pasa si falta)

**La regla:** `return` entrega un valor que puedes **guardar en una variable** o usar de inmediato. No todas las funciones lo necesitan — algunas solo *hacen* algo (imprimir, modificar, guardar un archivo) sin necesidad de entregar nada.

#### ⚠️ Si no hay `return`, Python devuelve `None`

No es que "no devuelva nada" — devuelve explícitamente `None`, que significa *"hice mi trabajo, pero no dejo ningún resultado guardable"*.

```python
def saludar():
    print("Hola!")

text = saludar()   # se EJECUTA el print, pero la función no RETORNA nada
print(text)        # None
```

Salida:

```text
Hola!
None
```

**Cómo leerlo:** la primera línea (`Hola!`) sale porque el `print()` de ADENTRO se ejecuta. La segunda línea (`None`) sale porque `text` nunca recibió un valor real — recibió el `None` automático.

#### 🔁 Con `return` vs sin `return` (comparación directa)

```python
# CON return — entrega el valor, se puede guardar y reutilizar
def format_name(name):
    name_lower = name.strip().lower()
    return name_lower

resultado = format_name("  ANA  ")
print(resultado)   # "ana"  → sí se guardó

# SIN return — solo imprime, no hay nada que guardar
def mostrar_saludo(nombre):
    print("Hola " + nombre)

resultado = mostrar_saludo("Ana")   # imprime "Hola Ana"
print(resultado)                     # None — no se guardó nada
```

⚠️ **Por qué esto importa para limpieza de datos (Error #2 de tu lista):** si escribes una función de limpieza SIN `return` y luego intentas asignarla a una columna…

```python
df["customer_age"] = clean_age(df["customer_age"])   # si clean_age no tiene return…
# ¡toda la columna se llena de None!
```

…pierdes la columna entera, no solo el valor. Para limpieza de datos, **casi siempre necesitas `return`**, porque el objetivo es obtener el valor limpio de vuelta para asignarlo.

#### Diagrama mental (entrada → función → salida)

```text
"ANA"  →  format_name()  →  "ana"
```

La función es la máquina fija; la entrada cambia, la salida cambia con ella — pero solo si hay `return` que la entregue.

---

### 32.6 El segundo propósito de `return`: cortar la ejecución

Además de entregar un valor, `return` **termina la función inmediatamente**. Cualquier línea después de él, dentro de esa misma función, **nunca se ejecuta**.

```python
def procesar_edad(edad):
    return f"Edad procesada: {edad}"
    print("Esto nunca se imprime")  # ⚠️ código inalcanzable

resultado = procesar_edad(25)
print(resultado)
```

Salida:

```text
Edad procesada: 25
```

⚠️ **Por qué importa:** si alguna vez ves que una parte de tu función "no corre" y no marca error, revisa si hay un `return` **antes** de esa línea. Python no avisa del código inalcanzable, simplemente lo ignora.

#### Resumen con/sin `return`

| Situación | Resultado |
|---|---|
| Función sin `return` | Siempre devuelve `None`, aunque haga otras cosas (imprimir, modificar) |
| Función con `return` | Devuelve el valor que tú decides, y corta ahí la ejecución |

**Otro caso de sin-`return` (refuerzo del patrón de 32.5):**

```python
def elevar_al_cuadrado(x):
    print("El cuadrado de", x, "es", x * x)

elevar_al_cuadrado(4)     # imprime, no devuelve
resultado = elevar_al_cuadrado(-3)
print(resultado)          # None — mismo patrón de siempre
```

---

### 32.7 Funciones con múltiples parámetros — y la trampa del orden

Una función puede recibir **más de una entrada**. Ejemplo realista (limpiar y combinar dos campos, como `city` + `state`):

```python
def formatear_nombre(nombre, apellido):
    nombre_formateado = nombre.strip().title()
    apellido_formateado = apellido.strip().title()
    return f"El apellido de {nombre_formateado} es {apellido_formateado}"

print(formatear_nombre("  ana  ", "GARCÍA"))
print(formatear_nombre("carlos", "  lópez  "))
```

#### ⚠️ La trampa: el orden posicional importa

Python **no adivina** cuál argumento es cuál — asigna por **posición**. El primero que pasas llena el primer parámetro, sin importar el nombre de tu variable.

```python
print(formatear_nombre("rodríguez", "  MARÍA"))
# La función cree que "rodríguez" es el NOMBRE y "MARÍA" el APELLIDO
# porque así los pasaste — aunque la intención era al revés
```

Corregido, respetando el orden que espera la función (`nombre` primero, `apellido` después):

```python
print(formatear_nombre("  MARÍA", "rodríguez"))
```

💡 **Regla práctica:** al llamar una función con varios parámetros, siempre revisa el **orden en que se definieron** (`def formatear_nombre(nombre, apellido)`) y respeta ese orden al llamarla — no el orden que te parezca lógico.

#### Ejemplo numérico — dos parámetros, caso de ventas

```python
def calcular_descuento(precio, porcentaje_descuento):
    tasa_descuento = porcentaje_descuento / 100  # convertir % a decimal
    descuento = precio * tasa_descuento
    return precio - descuento

# primero precio, después descuento — en ese orden
print(calcular_descuento(100, 10))   # 90.0
print(calcular_descuento(250, 20))   # 200.0
print(calcular_descuento(50, 15))    # 42.5
```

Patrón común en análisis de ventas: la función encapsula la **regla** de negocio (cómo se calcula el descuento); tú solo le das la entrada correcta — o, más adelante, se la aplicas a una columna completa del DataFrame.

#### Otro ejemplo de un solo parámetro (limpieza de texto, aplicable a cualquier dataset)

```python
def estandarizar_nombre(nombre):
    return nombre.strip().lower()

print(estandarizar_nombre("  ANA  "))   # "ana"
print(estandarizar_nombre("LuIs  "))    # "luis"
print(estandarizar_nombre("  CARLA"))   # "carla"
```

Sin importar espacios, mayúsculas o formatos inconsistentes, la función entrega siempre una versión limpia y estandarizada — la misma lógica de la sección 16, pero empaquetada para reutilizar.

---

### Términos clave

Función · `def` · **Parámetro** (la entrada que varía, va en la definición) · **Argumento** (el valor concreto que pasas al llamarla) · Cuerpo indentado · `return` · `None` · `snake_case` · **Orden posicional** · Código inalcanzable · Llamar/invocar · Reutilización.

---

## 33. Bucles (`for` & `while`) e iterables en una colección

> Las funciones definen **qué hacer**. Los bucles definen **a cuántos elementos se aplica**. Juntos son el motor de la automatización de limpieza.

**Cuándo usarlo:** cuando necesitas aplicar la misma regla —contar nulos, buscar sentinels, validar tipos— sobre varias columnas o valores. Sin bucle, tendrías que definir una función distinta por columna — el mismo problema de código repetido que resolviste con la sección 32.

---

### 33.1 ¿Qué es un bucle?

🏭 **La analogía de la banda transportadora:** cada producto pasa por la banda y se le aplica la **misma inspección**. En Python, lo que "pasa por la banda" puede ser:

- Una **lista**: `["ropa", "calzado", "accesorios"]`
- Un **string** (colección de caracteres): `"DATA"`

**Para qué sirven en limpieza de datos:**

- Revisar varias columnas y contar nulos
- Buscar sentinels (`999`, `"?"`) en distintas columnas — conecta con la sección 30.3
- Aplicar una función de limpieza (sección 32) a cada valor de una lista
- Construir listas de flags (1/0) según condiciones — conecta con tu bandera `edad_vacia` de la sección 31.2

---

### 33.2 Bucle `for` — el más usado en limpieza

**Sintaxis básica:**

```python
for elemento in coleccion:
    # hacer algo con elemento
```

| Parte | Qué es |
|---|---|
| `for` | Anuncia el inicio del bucle |
| `elemento` | Variable temporal que **cambia** en cada vuelta — tú eliges el nombre, casi cualquiera sirve |
| `in coleccion` | La lista/string/colección que se recorre |
| `:` | Cierra la línea — igual que en `def` (sección 32) |
| Línea indentada | El cuerpo: lo que se repite en cada vuelta |

**Ejemplo — iterar sobre las columnas del dataset:**

```python
columnas_a_revisar = ["product_category", "customer_segment", "city"]

for col in columnas_a_revisar:
    print("Revisando:", col)
```

Salida:

```text
Revisando: product_category
Revisando: customer_segment
Revisando: city
```

**Cómo leerlo, vuelta por vuelta:**

1. `col` toma el valor `"product_category"` → se ejecuta el cuerpo
2. `col` toma el valor `"customer_segment"` → se ejecuta el cuerpo otra vez
3. `col` toma el valor `"city"` → última vuelta

Es el **mismo bloque de código**, repitiéndose sobre cada elemento.

⚠️ **El nombre de la variable es libre** — `col`, `x`, `elemento`… todos funcionan igual. Pero un nombre representativo (`col` en vez de `x`) hace el código más fácil de leer después.

```python
# Funciona igual, pero es menos claro:
for x in columnas_a_revisar:
    print("Revisando:", x)
```

**Ejemplo — lista de productos:**

```python
productos = ["ropa", "calzado", "accesorios"]

for p in productos:
    print("Procesando:", p)
```

Salida:

```text
Procesando: ropa
Procesando: calzado
Procesando: accesorios
```

**Patrón:** una lista + una variable temporal + una acción repetida.

---

### 33.3 De código repetido a bucle — el caso de contar nulos

**Antes (repitiendo la línea a mano):**

```python
import pandas as pd
df = pd.read_csv("data/ventas_crudo.csv")

print("product_category nulos:", df["product_category"].isna().sum())
print("quantity nulos:", df["quantity"].isna().sum())
print("city nulos:", df["city"].isna().sum())
```

**Después (con bucle):**

```python
columnas = ["product_category", "quantity", "city"]

for col in columnas:
    print(col, "nulos:", df[col].isna().sum())
```

💡 **Por qué esto escala:** si mañana el dataset gana una columna nueva (`customer_region`), **solo agregas el nombre a la lista** — no reescribes la lógica:

```python
columnas = ["product_category", "customer_segment", "city", "customer_region"]
```

Esto es lo que en la sección 32 llamamos pasar de "código que funciona una vez" a "código que funciona siempre" — el bucle es la pieza que faltaba para aplicar esa idea a **columnas del DataFrame**, no solo a valores individuales.

**🔁 Sintaxis genérica (copiar y adaptar):**

```python
columnas_a_revisar = ["col_a", "col_b", "col_c"]

for col in columnas_a_revisar:
    print(col, "nulos:", df[col].isna().sum())
    # sustituye la línea de arriba por cualquier regla:
    # conteo de "?", validación de tipo, etc.
```

---

### 33.4 Bucle `while` — repetir mientras una condición sea verdadera

**Sintaxis básica:**

```python
while condicion:
    # código que se repite
```

A diferencia de `for` —que recorre una colección ya definida, con un número de vueltas fijo—, `while` repite **mientras una condición sea verdadera**, sin saber de antemano cuántas vueltas dará.

**Cuándo usarlo:**

- No sabes cuántas repeticiones habrá
- Repites hasta que se cumpla una condición
- Necesitas un contador manual o un proceso "hasta que quede limpio"

**Ejemplo:**

```python
contador = 3

while contador > 0:
    print("Contador:", contador)
    contador = contador - 1
```

Salida:

```text
Contador: 3
Contador: 2
Contador: 1
```

**Cómo leerlo, vuelta por vuelta:** Python evalúa `contador > 0` **antes** de cada vuelta. Si es verdadero, ejecuta el cuerpo; si es falso, se detiene.

1. `contador = 3` → `3 > 0` es verdadero → imprime, luego resta 1 → `contador = 2`
2. `2 > 0` es verdadero → imprime, resta 1 → `contador = 1`
3. `1 > 0` es verdadero → imprime, resta 1 → `contador = 0`
4. `0 > 0` es **falso** → el bucle termina, sin imprimir nada más

⚠️ **El riesgo real de `while`: el bucle infinito.** Si olvidas la línea `contador = contador - 1`, la condición `contador > 0` nunca cambia — sigue siendo verdadera para siempre y el código no termina nunca. Esta es la razón concreta por la que `while` "requiere más cuidado" que `for`: en `for`, la colección se agota sola; en `while`, **tú eres responsable** de que la condición eventualmente se vuelva falsa.

💡 **Checklist antes de correr un `while`:** ¿hay una línea dentro del cuerpo que modifica la variable de la condición? Si no la hay, no lo corras — vas a tener que interrumpir la ejecución a la fuerza.

💡 **Guía práctica del analista:** en limpieza y análisis de datos se usa mucho más `for` porque es más seguro y predecible — recorre una colección conocida y termina sola. `while` se deja para casos específicos ("hasta que quede limpio", contadores manuales), siempre revisando bien que el bucle vaya a terminar.

#### `for` vs `while` — cuál usar

| Situación | Usa |
|---|---|
| Ya sabes la colección completa (lista de columnas, valores de una fila) | `for` |
| No sabes cuántas vueltas harán falta; depende de una condición | `while` |

---

### 33.5 Cómo usan bucles los analistas — funciones + bucles combinados

**Idea central:** un analista rara vez limpia una sola columna o un solo valor. El patrón real es **encapsular la regla en una función (sección 32) y aplicarla dentro de un bucle (33.2–33.4)** sobre múltiples columnas o valores a la vez.

#### Caso 1 — Normalizar texto: función + `for` sobre una lista

```python
def limpiar_cadenas(lista_de_texto):
    for texto in lista_de_texto:
        print(texto.strip().title())

nombres_crudos_1 = ["  ANA  ", "LuIs ", " CARLA"]
nombres_crudos_2 = ["  ANA", " CArlos ", "Daniel ", "TanIA"]

limpiar_cadenas(nombres_crudos_1)
print()
limpiar_cadenas(nombres_crudos_2)
```

Salida:

```text
Ana
Luis
Carla

Ana
Carlos
Daniel
Tania
```

La función define la **regla** (`strip()` + `title()`); el bucle define **a cuántos elementos** se aplica. La misma función sirve para ciudades, categorías o segmentos de cliente — solo cambia la lista de entrada.

#### Caso 2 — Buscar sentinels en varias columnas

Retoma la sección 30.3 (sentinels): en vez de una celda de código por columna, un `for` lo resuelve en un solo bloque.

```python
columnas = ["customer_age", "quantity", "order_value"]
sentinel = -999

for col in columnas:
    print(col, "→", df[col].isin([-999]).sum())
```

Salida:

```text
customer_age → 25
quantity → 0
order_value → 0
```

**Lectura por iteración:** en la primera vuelta `col = "customer_age"` y se cuenta el `-999` en esa columna; en la segunda, `col = "quantity"`; en la tercera, `col = "order_value"`. Mismo patrón de la sección 33.3, aplicado ahora a detección de sentinels en vez de conteo de nulos.

💡 Si mañana quieres inspeccionar `discount` o `shipping_cost`, solo agregas el nombre a `columnas` — la lógica no cambia.

#### Caso 3 — Aplicar limpieza vectorizada a varias columnas de texto

```python
columnas_texto = ["city", "state", "product_category"]

for col in columnas_texto:
    df[col] = df[col].str.strip().str.lower()
```

En cada vuelta, `col` toma el nombre de una columna (`"city"`, luego `"state"`, luego `"product_category"`) y se le aplica `.str.strip().str.lower()` — el mismo limpiador básico de la sección 16, pero ahora sobre **tres columnas a la vez** en lugar de una por una.

📌 **Los 3 casos comparten un solo patrón:** define la regla una vez (función o línea de pandas), mete la lista de "a qué se aplica" en una variable, recorre con `for`. Cambiar el alcance de la limpieza es **agregar o quitar un nombre de la lista**, nunca reescribir lógica.

---

### Recursos adicionales

- [`for`](https://docs.python.org/3/tutorial/controlflow.html#more-control-flow-tools)[, ](https://docs.python.org/3/tutorial/controlflow.html#more-control-flow-tools)[`while`](https://docs.python.org/3/tutorial/controlflow.html#more-control-flow-tools)[, ](https://docs.python.org/3/tutorial/controlflow.html#more-control-flow-tools)[`break`](https://docs.python.org/3/tutorial/controlflow.html#more-control-flow-tools)[, ](https://docs.python.org/3/tutorial/controlflow.html#more-control-flow-tools)[`continue`](https://docs.python.org/3/tutorial/controlflow.html#more-control-flow-tools)[, ](https://docs.python.org/3/tutorial/controlflow.html#more-control-flow-tools)[`else`](https://docs.python.org/3/tutorial/controlflow.html#more-control-flow-tools)[ — Instrucciones de control](https://docs.python.org/3/tutorial/controlflow.html#more-control-flow-tools)
- [Listas y operaciones básicas](https://docs.python.org/3/tutorial/introduction.html#lists)
- [Strings como secuencias iterables](https://docs.python.org/3/library/stdtypes.html#text-sequence-type-str)

📌 **Hacia dónde va esto:** combinar bucles + funciones para construir una librería de limpieza reutilizable y aplicarla a un dataset real. Es exactamente lo que arman las secciones 34 y 35.

### Términos clave

Bucle / Loop · `for` · `while` · Iterable (lista, string) · Colección · Variable temporal · Iteración / vuelta · Cuerpo del bucle · Condición · Bucle infinito · Contador · **Automatización de limpieza** (funciones + bucles aplicando reglas de forma masiva y consistente).

---

## 34. Mini-funciones de limpieza — un loop DENTRO de una función

> El puente entre sección 32 (funciones) y sección 33 (bucles): meter un `for` **dentro** de una función para que una sola regla se aplique a cualquier número de columnas.

**Cuándo usarlo:** cuando varias columnas necesitan exactamente el mismo tipo de limpieza (convertir a numérico, estandarizar texto, etc.) y quieres que agregar una columna nueva sea tan simple como agregar un nombre a una lista — sin tocar la lógica.

📘 Ya viste una versión de este patrón en la sección 33.5 (Caso 1, `limpiar_cadenas`). Esta lección lo formaliza y — lo más valioso— explica **por qué la alternativa obvia (una función por columna) NO es la solución**.

---

### 34.1 El problema: repetición sin loop

Convertir tres columnas a numérico, a mano, columna por columna:

```python
df["price"] = pd.to_numeric(df["price"], errors="coerce")
df["quantity"] = pd.to_numeric(df["quantity"], errors="coerce")
df["order_value"] = pd.to_numeric(df["order_value"], errors="coerce")
```

**Por qué no escala:**

- Añadir una columna nueva = escribir otra línea
- Cambiar la regla (ej. otro parámetro) = cambiarla en **cada** línea, una por una

Conecta directo con la sección 18 (`pd.to_numeric(..., errors="coerce")`) — aquí es la misma herramienta, el problema es que la estás repitiendo a mano.

---

### 34.2 El intento intermedio (antipatrón): una función por columna

⚠️ **Esta subsección es la más valiosa del tema: NO copies este patrón, pero entiende por qué falla.**

La idea "obvia" para evitar repetición: envolver cada línea en su propia función.

```python
def convertir_price(df):
    df["price"] = pd.to_numeric(df["price"], errors="coerce")
    return df

def convertir_quantity(df):
    df["quantity"] = pd.to_numeric(df["quantity"], errors="coerce")
    return df

def convertir_order_value(df):
    df["order_value"] = pd.to_numeric(df["order_value"], errors="coerce")
    return df

df = convertir_price(df)
df = convertir_quantity(df)
df = convertir_order_value(df)
```

**Se ve más ordenado — pero no resuelve nada:**

| Problema | Por qué sigue ahí |
|---|---|
| Código repetido | Las tres funciones contienen **la misma línea** (`pd.to_numeric(df[col], errors="coerce")`) copiada tres veces |
| No escalable | Si el dataset gana `"discount"` o `"shipping_fee"`, necesitas **otra función más** |
| Mantenimiento frágil | Si la regla cambia (otro método de conversión), hay que actualizar **todas** las funciones, una por una — fácil olvidar alguna |

💡 **La lección de fondo:** envolver código repetido en funciones separadas NO elimina la repetición — solo la reorganiza. Repetición real se elimina cuando la **regla vive en un solo lugar** y varía únicamente la **entrada**.

---

### 34.3 La solución: una función con un loop interno

```python
def convertir_columnas_numericas(df, columnas):
    for col in columnas:
        df[col] = pd.to_numeric(df[col], errors="coerce")
    return df
```

**Qué cambió:** en vez de una función por columna, hay **una sola función** que recibe la lista completa de columnas como parámetro. El `for` de adentro (sección 33.2) aplica la misma regla a cada elemento de esa lista.

- La **función** define la regla (`pd.to_numeric` con `errors="coerce"`)
- El **loop** decide a cuántas columnas se aplica — exactamente el principio de la sección 33 ("las funciones definen qué hacer, los bucles definen a cuántos elementos")

**Uso:**

```python
columnas_numericas = ["price", "quantity", "order_value"]

df = convertir_columnas_numericas(df, columnas_numericas)
df.info()
```

Una sola línea convierte las tres columnas. `df.info()` las muestra ya en formato numérico.

#### Escalar con un cambio de una línea

```python
columnas_numericas.append("customer_age")
df = convertir_columnas_numericas(df, columnas_numericas)
df.info()
```

Para sumar `customer_age` a la conversión, **no se reescribe ninguna función** — solo se agrega el nombre a la lista con `.append()`. El loop hace el resto.

**🔁 Sintaxis genérica (copiar y adaptar):**

```python
def aplicar_regla_a_columnas(df, columnas):
    for col in columnas:
        df[col] = # tu regla de limpieza aquí (to_numeric, str.strip(), fillna, etc.)
    return df

lista_de_columnas = ["col_a", "col_b"]
df = aplicar_regla_a_columnas(df, lista_de_columnas)

# Escalar: agregar una columna sin tocar la función
lista_de_columnas.append("col_c")
df = aplicar_regla_a_columnas(df, lista_de_columnas)
```

---

### 34.4 Mini-reflexión — el costo real de escalar

**Pregunta:** si mañana el dataset gana `"shipping_fee"`, `"tax_amount"` y `"total_discount"`, ¿cuánto código nuevo hace falta?

**Respuesta — con el patrón correcto (34.3), una sola línea:**

```python
columnas_numericas = ["price", "quantity", "order_value"]
columnas_numericas.append("shipping_fee")
# la función y el loop hacen el resto
```

Compara este costo con el patrón del 34.2 (una función por columna): ahí cada columna nueva significaba **una función completa** que escribir. Ese contraste es la prueba de que el patrón "función + loop interno" es el que realmente escala.

---

### Comparación de los tres enfoques

| Enfoque | Código repetido | Agregar columna nueva | Cambiar la regla |
|---|---|---|---|
| Líneas sueltas (34.1) | Sí, una línea por columna | Escribir otra línea | Editar cada línea |
| Función por columna (34.2) | Sí, misma lógica copiada en cada función | Escribir otra función | Editar cada función |
| Función + loop interno (34.3) | No — la regla vive en un solo lugar | Agregar un nombre a la lista | Editar una sola línea, dentro de la función |

### Términos clave

Función (define la regla) · Loop dentro de la función (repite la regla sobre muchas columnas) · Escalabilidad (cambiar la lista cambia el resultado sin reescribir código) · Antipatrón · `.append()`.

---

📌 **Siguiente paso:** encadenar varias de estas funciones en un **pipeline completo** de limpieza (sección 35).

---

## 35. Pipeline completo de limpieza — de mini-funciones a `clean_data(df)`

> El cierre de las secciones 32–34: encadenar varias funciones pequeñas en una sola función orquestadora que recibe el DataFrame crudo y entrega el limpio, en un solo comando.

**Cuándo usarlo:** cuando ya diagnosticaste el dataset (sección 30) y decidiste las estrategias por columna (sección 31) — este es el paso que convierte esas decisiones en un programa reutilizable, en vez de código suelto que se repite cada vez que llega un dataset nuevo.

---

### 35.1 Qué es un pipeline

Un pipeline de limpieza es una línea de producción: entra el dataset crudo → pasa por etapas ordenadas (texto → tipos → inválidos/faltantes) → sale un dataset limpio, listo para analizar.

```python
def limpiar_datos(df):
    df = paso_limpia_texto(df)
    df = paso_corrige_edad(df)
    df = paso_convierte_tipos(df)
    return df
```

Cada paso recibe un DataFrame, aplica una función, y **devuelve el DataFrame actualizado** — el mismo contrato de `return` que viste en la sección 32.5, encadenado varias veces.

---

### 35.2 Función 1 — `reemplazar_sentinels_global` (sentinels + inválidos)

Retoma el diagnóstico de 30.3: `customer_age` usaba `-999`/`999` como sentinel numérico; `product_category` usaba `"?"` como sentinel de texto (sin `NaN`s reales).

⚠️ **Verifica que la función realmente corrió (ver 37.5).** El fallo típico: el pipeline queda escrito, pero al revisar el archivo limpio `df["product_category"].value_counts()` **todavía muestra `?`**. O el archivo que estás leyendo no pasó por esta función, o la lista `columnas_texto` no cubrió esa columna. Comprueba la salida antes de dar por buena una limpieza.

**Código base:**

```python
df["customer_age"] = df["customer_age"].replace(-999, pd.NA)
df["product_category"] = df["product_category"].replace("?", "unknown")
```

**Convertido en función reutilizable con dos bucles internos** (patrón de la sección 34.3: función + `for` adentro):

```python
def reemplazar_sentinels_global(df, numeric_cols, text_cols):
    numeric_sentinels = [-999, 999]
    for col in numeric_cols:
        df[col] = pd.to_numeric(df[col], errors="coerce")
        df[col] = df[col].replace(numeric_sentinels, pd.NA)

    text_sentinels = ["?"]
    for col in text_cols:
        df[col] = df[col].replace(text_sentinels, "unknown")
    return df
```

- Primer bucle: convierte cada columna numérica y reemplaza sus sentinels por `pd.NA` (para poder tratarlos después con tu flujo de nulos, secciones 6/9/31).
- Segundo bucle: reemplaza sentinels de texto directo por `"unknown"` — aquí no hace falta pasar por `NaN` primero porque no hay ambigüedad de tipo.

**Uso:**

```python
columnas_numericas = ["customer_age"]
columnas_texto = ["product_category"]

df = reemplazar_sentinels_global(df, columnas_numericas, columnas_texto)
```

⚠️ **Error clásico al copiar código de ejemplo:** aparece `reemplazar_sentinels_global(df, columnas_numericas, columnas_texto):` con dos puntos al final. Eso es sintaxis de `def`, no de una llamada a función — con los dos puntos ahí, Python marca `SyntaxError`. Al llamar una función (no definirla) nunca lleva `:`.

---

### 35.3 Función 2 — `crear_flags` (banderas antes de imputar)

Retoma 31.2/31.3: antes de imputar, conserva evidencia de qué filas eran originalmente vacías (`edad_vacia` ya lo hiciste para una sola columna; aquí se generaliza a varias).

```python
def crear_flags(df, flags_cols):
    for col in flags_cols:
        nombre_flag = col + "_missing_flag"
        df[nombre_flag] = df[col].isna().astype(int)
    return df
```

**Cómo leer `nombre_flag = col + "_missing_flag"`:** en la primera vuelta `col = "customer_age"`, y se concatena con el texto `"_missing_flag"` → `"customer_age_missing_flag"`. Es concatenación de strings con `+` (no `f-string` aquí), y sirve para generar nombres de columna dinámicos dentro del bucle — algo que no habías hecho antes: el *nombre de la columna nueva* también se construye dentro del loop, no solo su contenido.

**Uso:**

```python
columnas_flags = ["customer_age", "city", "state"]
df = crear_flags(df, columnas_flags)
```

⚠️ **Inconsistencia frecuente en código copiado:** la función se define como `crear_flags(df, flags_cols)` pero más abajo se llama como `crear_flags_antes_de_imputar(df, columnas_flags)` — un nombre que nunca se definió. Si lo copias tal cual, Python lanza `NameError: name 'crear_flags_antes_de_imputar' is not defined`. Usa siempre el nombre exacto con el que hiciste el `def`.

---

### 35.4 Función 3 — `imputar_segun_diagnostico` (imputación por tipo de columna)

Combina tres estrategias distintas, cada una para un tipo de columna, según lo que ya diagnosticaste en la sección 31:

| Tipo de columna | Diagnóstico (dataset de ejemplo) | Estrategia |
|---|---|---|
| Numérica (`customer_age`) | MCAR, \~3%, no depende de otra variable | Imputar con mediana (sección 31.3) |
| Categórica (`city`, `state`) | MCAR uniforme, \~2%, sin razón de negocio para eliminar | Imputar con `"unknown"` |
| Fecha (`order_date`) | MCAR, \~0.24%, imputar inventaría registros falsos | Eliminar filas (`dropna`) |

```python
def imputar_segun_diagnostico(df, median_fill_cols, fill_unknown_cols, date_drop_cols):
    # rellenar con la mediana en columnas numéricas
    for col in median_fill_cols:
        df[col] = pd.to_numeric(df[col], errors="coerce")
        med = df[col].median()
        df[col] = df[col].fillna(med)

    # rellenar con texto "unknown" en columnas categóricas
    for col in fill_unknown_cols:
        df[col] = df[col].fillna("unknown")

    # eliminar registros con valores ausentes en columnas tipo fecha
    for col in date_drop_cols:
        df[col] = pd.to_datetime(df[col], errors="coerce")
        df = df.dropna(subset=[col]).reset_index(drop=True)
    return df
```

💡 **Por qué fecha se elimina y no se imputa:** a diferencia de `customer_age` (mediana) o `city`/`state` (`"unknown"`), inventar una fecha de pedido falsa distorsionaría cualquier análisis de estacionalidad o serie temporal — no hay un valor "neutral" creíble para una fecha. Con solo \~0.24% de filas afectadas, el costo de eliminarlas es mínimo.

**Uso:**

```python
cols_imputar_mediana = ["customer_age"]
cols_imputar_unknown = ["city", "state"]
cols_imputar_fecha = ["order_date"]

df = imputar_segun_diagnostico(df, cols_imputar_mediana, cols_imputar_unknown, cols_imputar_fecha)
```

---

### 35.5 El pipeline completo — `clean_data(df)`

**Idea central:** `clean_data()` no contiene lógica de limpieza propia — solo **orquesta**, llamando a las tres funciones anteriores en el orden correcto (texto/sentinels → flags → imputación). La lógica detallada vive en las funciones pequeñas; esto facilita probar y mantener cada pieza por separado.

```python
def clean_data(df, numeric_cols, text_cols, flags_cols,
                median_fill_cols, fill_unknown_cols, date_drop_cols):

    # Sentinels: reemplazar marcadores inválidos por NaN
    df = reemplazar_sentinels_global(df, numeric_cols, text_cols)

    # Flags: crear banderas antes de imputar
    df = crear_flags(df, flags_cols)

    # Imputaciones / drops finales
    df = imputar_segun_diagnostico(df, median_fill_cols, fill_unknown_cols, date_drop_cols)
    return df
```

📌 **Por qué el orden importa:** los flags (35.3) se crean **antes** de imputar (35.4) — si imputaras primero, ya no habría `NaN`s que marcar y perderías la evidencia de qué filas eran originalmente vacías. Mismo principio de "ausencia como señal" de la sección 31.1.

---

### 35.6 `main_pipeline.py` — cargar, limpiar, exportar

Plantilla genérica (llena las listas de columnas según el dataset):

```python
# main_pipeline.py
df = pd.read_csv(#ruta al DF aquí)

## columnas función 1: reemplazar_sentinels
columnas_numericas = []
columnas_texto = []

## columnas función 2: crear_flags
columnas_flags = []

## columnas función 3: imputar_segun_diagnostico
cols_imputar_mediana = []
cols_imputar_unknown = []
cols_imputar_fecha = []

df_clean = clean_data(df, columnas_numericas, columnas_texto, columnas_flags,
    cols_imputar_mediana, cols_imputar_unknown, cols_imputar_fecha)

df_clean.to_csv("RUTA_AQUI/nombre_archivo.csv", index=False)
```

**Aplicado al dataset de ejemplo:**

```python
# main_pipeline.py
df = pd.read_csv("data/ventas_crudo.csv")

columnas_numericas = ["customer_age"]
columnas_texto = ["product_category"]
columnas_flags = ["customer_age", "city", "state"]
cols_imputar_mediana = ["customer_age"]
cols_imputar_unknown = ["city", "state"]
cols_imputar_fecha = ["order_date"]

df_clean = clean_data(df, columnas_numericas, columnas_texto, columnas_flags,
    cols_imputar_mediana, cols_imputar_unknown, cols_imputar_fecha)

df_clean.to_csv("data/ventas_limpio.csv", index=False)
```

Retoma la sección 29 (`.to_csv(..., index=False)`) para el paso final de exportación.

---

### Flujo completo de la sección 35 (orden importa)

1. Diagnostica (sección 30) y decide estrategia por columna (sección 31) — **antes** de escribir cualquier función.
2. Construye las mini-funciones una por una: `reemplazar_sentinels_global` → `crear_flags` → `imputar_segun_diagnostico`.
3. Encadénalas dentro de `clean_data(df, ...)`, respetando el orden: sentinels → flags → imputación.
4. Define las listas de columnas específicas del dataset en `main_pipeline.py`.
5. Ejecuta, valida (`df_clean.info()`, `.isna().sum()` — secciones 10/15) y exporta con `.to_csv(index=False)`.

### ⚠️ Errores a vigilar (de esta sección)

- No pongas `:` al final de una **llamada** a función — solo al **definirla** con `def`.
- El nombre con el que llamas a una función debe ser **idéntico** al que usaste en su `def`; un nombre inventado da `NameError`.
- Crea los flags (35.3) **antes** de imputar (35.4), nunca después.

### Términos clave

Pipeline de limpieza · Orden de operaciones de limpieza (texto → tipos → valores inválidos) · Función orquestadora (`clean_data`, no contiene lógica propia, solo llama a otras) · `main_pipeline.py` · Concatenación de strings para nombres dinámicos (`col + "_missing_flag"`).

---

📌 **Siguiente capítulo:** análisis de distribuciones para responder — ¿qué patrones iniciales revelan sobre el comportamiento, distribución y valor de los clientes?

---

## 43. Lógica condicional — `if` / `else`

**Cuándo usarlo:** cuando el código tiene que **tomar una decisión**: hacer una cosa u otra según el valor de un dato. Es la base del *feature engineering* — crear columnas nuevas a partir de reglas de negocio (segmentos, niveles de valor, indicadores de riesgo).

📌 **Es la tercera estructura de control de esta guía**, junto a las funciones (sección 32) y los bucles (sección 33). Se combinan: un `if` **dentro** de una función, dentro de un `for`, es el patrón con el que se construye una columna de segmentos.

### 43.1 La estructura

```python
# Solo if — si no se cumple, Python se salta el bloque y sigue abajo
valor_venta = 150

if valor_venta > 100:
    print("Venta alta")

print("Código terminado")   # esto se ejecuta SIEMPRE (está fuera del if)
```

```python
# if / else — siempre se ejecuta una de las dos ramas
valor_venta = 50

if valor_venta > 100:
    print("Venta alta")
else:
    print("Venta baja")
```

| Parte | Qué hace |
|---|---|
| `if condición:` | Evalúa la condición. Si es `True`, ejecuta el bloque indentado |
| `else:` | **Opcional.** Qué hacer cuando la condición fue `False`. No lleva condición propia |
| Bloque indentado | Todo lo que va con sangría pertenece a esa rama |
| Código sin indentar debajo | Se ejecuta **siempre**, pase lo que pase con el `if` |

🎯 **`if` sin `else` no es un error.** Si la condición es falsa simplemente no pasa nada y el programa continúa. Usa `else` solo cuando necesites una acción alternativa real.

### 43.2 ⚠️ Los dos puntos y la indentación — tus dos errores recurrentes

Esta estructura reúne **exactamente** los dos fallos que ya has cometido en `for` y en `def`:

```python
# ❌ Falta el :
if valor_venta > 100
    print("Venta alta")
# SyntaxError: expected ':'

# ❌ Falta la indentación
if valor_venta > 100:
print("Venta alta")
# IndentationError: expected an indented block
```

💡 **Truco de diagnóstico que te ahorra tiempo:** suele decirse que "al dar Enter aparece la indentación automática". Cierto, **pero solo aparece si escribiste bien los dos puntos**. Así que si presionas Enter y el cursor **no** se movió a la derecha, no es que el editor falle: **te faltó el `:`**. La indentación automática funciona como verificador gratuito de la línea anterior.

📌 Los dos puntos + bloque indentado son idénticos en `if`, `else`, `for`, `while` y `def` (secciones 32.3 y 33.2). Es la misma gramática en cinco lugares: **encabezado, dos puntos, bloque con sangría**.

### 43.3 Operadores de comparación

| Operador | Significa | Ejemplo |
|---|---|---|
| `>` / `<` | Mayor / menor que | `if precio > 100:` |
| `>=` / `<=` | Mayor o igual / menor o igual | `if cantidad <= 0:` |
| `==` | **Igual a** (comparación) | `if ciudad == "Houston":` |
| `!=` | Distinto de | `if categoria != "?":` |
| `and` / `or` | Combinar condiciones | `if precio > 100 and cantidad > 5:` |

⚠️ **`=` no es `==`.** Un solo signo **asigna** un valor; dos signos **comparan**. `if x = 5:` es `SyntaxError`. Es el error más común al empezar con condicionales.

💡 Los operadores son los mismos que ya usas para filtrar filas en la sección 7 (`df[df["col"] > 100]`) y en la asignación condicional de la 31.4 (`df.loc[cond, col] = valor`). Lo que cambia es **sobre qué** se aplican: ahí sobre una columna entera, aquí sobre **un solo valor**.

### 43.4 ⚠️ Un `if` NO funciona sobre una columna

Este es el error que aparece en cuanto pasas del ejemplo con un solo valor a un DataFrame real:

```python
# ❌ Esto revienta
if df["price"] > 100:
    print("caro")
```

```text
ValueError: The truth value of a Series is ambiguous.
Use a.empty, a.bool(), a.item(), a.any() or a.all()
```

🎯 **Por qué:** `if` necesita **un solo** `True` o `False` para decidir. `df["price"] > 100` no devuelve uno: devuelve **5,000** valores `True`/`False`, uno por fila. Python no sabe cuál usar, así que se detiene.

| Contexto | Qué usar | Dónde |
|---|---|---|
| **Un valor suelto** (un número, un parámetro de función) | `if` / `else` | Esta sección |
| **Una columna entera**, asignar un valor a las filas que cumplen | `df.loc[condición, "columna"] = valor` | Sección 31.4 |
| **Una columna entera**, quedarte solo con las filas que cumplen | `df[df["columna"] > 100]` | Secciones 7 y 21 |

💡 **La forma correcta de usar `if` con un DataFrame** es escribir la regla para **un valor** dentro de una función, y dejar que pandas la aplique fila por fila:

```python
def clasificar_venta(valor):
    if valor > 100:
        return "alta"
    else:
        return "baja"

# La función recibe UN valor cada vez, así que el if sí funciona
clasificar_venta(150)   # → "alta"
```

Ese es el puente entre esta sección y la 32: **la función aísla un valor, y dentro de la función el `if` ya es válido.**

### 43.5 Lo que viene

La lección se titula "Segmentación de clientes" pero por ahora solo cubre `if`/`else` sobre un valor suelto y con `print()`. Faltan dos piezas que llegarán después:

- **`elif`** — ya está desarrollado abajo, en la **43.6**.
- **Aplicar la regla a toda una columna** para crear la columna de segmento. Las herramientas habituales son `.apply()`, `np.where()` y `pd.cut()`.

⚠️ **Con `print()` no se segmenta nada.** `print` muestra texto en pantalla y lo pierde; para crear un segmento necesitas **`return`** dentro de una función (la sección 32.5) y guardar el resultado en una columna. Es la distinción `print` vs `return` que aparece — aquí es donde se vuelve crítica.

### 43.6 Más de dos casos — `elif`

`elif` (abreviatura de *else if*) encadena condiciones. Cada una se revisa **solo si todas las anteriores fallaron**.

```python
puntuacion = 7.5

if puntuacion >= 9:
    clasificacion = "Excelente"
elif puntuacion >= 7:
    clasificacion = "Buena"
elif puntuacion >= 5:
    clasificacion = "Regular"
else:
    clasificacion = "Mala"

print(clasificacion)   # → Buena
```

🎯 **Python evalúa en orden y se detiene en la primera condición verdadera.** Con 7.5: revisa `>= 9` → falso, sigue; revisa `>= 7` → **verdadero**, asigna "Buena" y **abandona toda la cadena** sin mirar el resto. Solo hay un ganador.

💡 **Por eso los rangos son implícitos y no hace falta escribirlos.** `elif puntuacion >= 7` significa en realidad *"entre 7 y 9"*, porque si fuera 9 o más ya habría salido en la línea anterior. No necesitas `elif 7 <= puntuacion < 9`.

#### ⚠️ La trampa del orden

Si ordenas las condiciones de menor a mayor, **las categorías altas se vuelven inalcanzables**:

```python
# ❌ ORDEN INCORRECTO — no da error, pero está mal
puntuacion = 9.8

if puntuacion >= 5:
    clasificacion = "Regular"     # ← 9.8 entra AQUÍ y se detiene
elif puntuacion >= 7:
    clasificacion = "Buena"       # ← inalcanzable
elif puntuacion >= 9:
    clasificacion = "Excelente"   # ← inalcanzable

print(clasificacion)   # → "Regular"  (¡con 9.8!)
```

**Nunca lanza un error.** Simplemente clasifica mal todo el dataset en silencio, y no lo notas hasta que alguien pregunta por qué no hay clientes Excelentes.

📘 **Regla:** con umbrales `>=`, ordena **de mayor a menor**. Con umbrales `<=`, de menor a mayor. Y valida siempre con `value_counts()` que **todas** las categorías aparezcan:

```python
df["segmento"].value_counts()   # ¿falta alguna categoría? → revisa el orden
```

#### ⚠️ El `else` es un cajón de sastre: los `NaN` caen ahí

Esto es lo más importante de la sección y casi nunca se menciona. En Python **cualquier comparación con `NaN` devuelve `False`**:

```python
float("nan") >= 9    # False
float("nan") >= 7    # False
float("nan") >= 5    # False
```

Así que un `NaN` **falla todas las condiciones y aterriza en el `else`**. Si aplicas esta función a una columna con nulos, **cada valor faltante queda clasificado como "Mala"** — la peor categoría — sin un solo aviso.

```python
# ✅ Blinda el else: trata los faltantes de forma explícita
def clasificar(puntuacion):
    if pd.isna(puntuacion):
        return "Sin dato"
    if puntuacion >= 9:
        return "Excelente"
    elif puntuacion >= 7:
        return "Buena"
    elif puntuacion >= 5:
        return "Regular"
    else:
        return "Mala"
```

📌 **Conecta con las secciones 6, 9 y 31:** ahí decides qué hacer con los nulos **antes** de segmentar. Si no lo hiciste, el `else` decide por ti — y decide mal.

⚠️ **El `else` tampoco filtra valores imposibles.** Con la cadena de arriba, una puntuación de **50** en una escala de 0 a 10 saldría como "Excelente", porque `50 >= 9` es verdadero. Valida el rango (secciones 14 y 30.3) antes de clasificar.

#### ⚠️ Los límites: `>` no es `>=`

De la práctica guiada: con `valor_venta = 100` y la condición `if valor_venta > 100:`, el resultado es **"Venta baja"**, porque 100 no es mayor que 100.

| Quiero que 100 sea "alta" | Escribo |
|---|---|
| Incluyendo el límite | `if valor_venta >= 100:` |
| Con el umbral una unidad abajo | `if valor_venta > 99:` (frágil con decimales: 99.5 también entra) |

💡 **Prueba siempre el valor exacto del umbral.** Los errores de segmentación casi nunca están en el centro de los rangos: están justo en la frontera. Es el mismo cuidado de la sección 31.4, donde `quantity <= 0` incluye el cero y `quantity < 0` lo dejaría pasar.

⚠️ **Comillas curvas.** Es común encontrar `print(”Valor”, valor_venta)` con comillas tipográficas (`”`) en vez de rectas (`"`). Copiado tal cual da `SyntaxError: invalid character`. Pasa seguido al copiar desde un documento o una página web — si un `print` falla sin motivo aparente, mira las comillas.

💡 **Renombrar una variable es renombrarla en TODAS partes.** Si `valor_venta` pasa a `cantidad_venta`, hay que cambiarla en la asignación **y** en la condición del `if`. Si olvidas una, obtienes `NameError` — o peor, si la vieja variable aún existe, el código corre con el valor equivocado sin avisar.

### 43.7 ⚠️ Caso práctico — clasificar un promedio **no** es segmentar

Es común ver `if`/`elif` aplicado al **promedio** de una columna:

```python
edad_promedio = df['customer_age'].mean()      # 49.12

if edad_promedio > 55:
    print("Clientes senior")
else:
    print("Clientes junior")                    # → se imprime esto

gasto_promedio = df['order_value'].mean()      # 10,075.52

if gasto_promedio > 10000:
    print("Clientes: VIP")                      # → se imprime esto
elif gasto_promedio > 5000:
    print("Clientes: Medium Value")
else:
    print("Clientes: Low Value")
```

El código funciona. **La conclusión que se saca de él, no.**

#### El problema de fondo

🎯 **Estás clasificando UN número, no a tus clientes.** El `if` recibe un solo escalar — el promedio — así que el resultado describe **al promedio**, no a la base de clientes. Concluir *"predominan clientes VIP"* a partir de una sola cifra es un salto lógico: para decir quién predomina hay que **contar clientes**, no evaluar un agregado.

#### Cuatro razones concretas por las que falla aquí

**1. El razonamiento se muerde la cola.** El argumento suele ser: *"la media supera los 10000, podemos concluir que predominan clientes VIP, cuyos altos gastos elevan la media"*. Si la media está inflada por unos pocos, entonces esos pocos **no** predominan — es exactamente el argumento contrario. Es la lógica de la sección 36.3 al revés.

**2. Además, esa explicación es falsa para esta columna.** En `order_value` la media (10,075) está **por debajo** de la mediana (10,341), según la tabla de 36.7. La cola alta **no** está inflando la media aquí; la columna es bimodal (40.2). La frase se copió del caso típico de sesgo a la derecha sin comprobarla contra los datos.

**3. El umbral "VIP" cae casi exactamente en la mediana.** Con mediana 10,341, **más de la mitad de los pedidos superan los 10,000**. Una etiqueta VIP que abarca a más del 50% de tu base no segmenta nada — solo parte el dataset por la mitad y le pone un nombre aspiracional.

**4. La conclusión es frágil hasta lo absurdo.** La media es 10,075.52 y el umbral 10,000: un margen de **0.75%**. Mueve el umbral a 10,100 — una decisión igual de arbitraria — y toda la "conclusión de negocio" se invierte. Un hallazgo que depende del tercer decimal de un umbral inventado no es un hallazgo.

#### Y `customer_age`, otra vez

`edad_promedio` = 49.12 → "Clientes junior". Pero esa columna es **uniforme entre 18 y 80** (36.7): su media es simplemente el punto medio, (18 + 80) / 2 = 49. La "conclusión" equivale a decir que el punto medio de 18 y 80 es menor que 55.

Y aquí se ve el daño de clasificar un agregado. Si la distribución es uniforme, la proporción de clientes realmente mayores de 55 es:

```text
(80 − 55) / (80 − 18) = 25 / 62 ≈ 40%
```

**Cuatro de cada diez clientes SÍ son senior.** El promedio los borró a todos con una sola etiqueta. Ese 40% es exactamente la información que una segmentación real debería revelar.

#### ✅ Lo que sí hay que hacer

Aplicar la regla **fila por fila** y después contar:

```python
# La lógica se escribe para UN valor (43.4), no para la columna
def segmentar_edad(edad):
    if pd.isna(edad):
        return "Sin dato"
    elif edad > 55:
        return "Senior"
    else:
        return "Junior"

# Y se aplica a toda la columna — esto es lo que verás en la próxima lección
df["segmento_edad"] = df["customer_age"].apply(segmentar_edad)

# Ahora sí puedes decir quién predomina
df["segmento_edad"].value_counts(normalize=True)
```

🎯 **La diferencia:** el `if` sobre el promedio devuelve **una palabra**; el `if` aplicado fila por fila devuelve **una columna**, y esa columna sí se puede contar, cruzar con otras variables y usar para decidir. Es el puente entre esta sección y tu 37 (`value_counts`, frecuencia relativa).

#### ⚠️ Cuándo SÍ tiene sentido clasificar un agregado

Para ser justos: evaluar un promedio con un `if` **es válido** en un contexto — el de **alerta o KPI**. Por ejemplo *"si el ticket promedio del mes cae por debajo de 8,000, avísame"*. Ahí la unidad de análisis **es** el mes, no el cliente.

Lo inválido es usar ese resultado para hacer afirmaciones sobre **la composición** de la base. *"El ticket promedio es de nivel VIP"* es correcto; *"predominan clientes VIP"* no se sigue de ahí.

📘 **Consejo que sí vale:** ordena las condiciones **de la más restrictiva a la menos restrictiva**, para que los casos especiales (errores, VIP) se capturen primero. Es la misma regla del orden de la 43.6, formulada de manera más general y aplicable también a umbrales no numéricos.

⚠️ **Detalle fácil de pasar por alto:** el comentario dice `# Clasificar clientes según el gasto mediano`, pero la variable es `gasto_promedio` y se calculó con `.mean()`. **Media y mediana no son lo mismo** (sección 22) — y en esta columna dan 10,075 vs 10,341, que caen a lados distintos de umbrales cercanos. Comentario y código deben decir lo mismo.

### Términos clave

`if` · `elif` · `else` · **Condición** (expresión que da `True` o `False`) · Bloque indentado · Dos puntos `:` · Operadores de comparación (`>`, `>=`, `==`, `!=`) · **`=` (asigna) vs `==` (compara)** · `and` / `or` · **Feature engineering** (crear columnas a partir de reglas de negocio) · `ValueError: truth value of a Series is ambiguous` · Valor escalar vs Serie · **Evaluación en cadena** (se detiene en la primera verdadera) · Rango implícito del `elif` · **Orden de los umbrales** (mayor a menor con `>=`) · Categoría inalcanzable · `else` como cajón de sastre · **`NaN` falla toda comparación y cae en el `else`** · `pd.isna()` · Valor frontera · Comillas curvas vs rectas · **Unidad de análisis** (el agregado vs la fila) · Clasificar un agregado ≠ segmentar · Sensibilidad al umbral · `.apply()` (aplicar una función fila por fila) · Condiciones ordenadas de más a menos restrictiva.

📌 **Conecta con:** sección 7 y 21 (filtrar filas con condición — los mismos operadores sobre columnas), 31.4 (asignación condicional con `.loc`), 32.3 y 32.5 (anatomía de una función, `print` vs `return`), 33.2 y 33.4 (dos puntos e indentación en `for` y `while`), 37 (segmentos y categorías).

---

[← Diagnóstico de calidad e imputación](06-calidad-de-datos.md) · [Índice](../README.md) · [Distribuciones e histogramas →](08-distribuciones.md)
