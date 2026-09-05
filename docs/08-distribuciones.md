# Distribuciones e histogramas

[← Índice de la guía](../README.md)

---

## 36. Diagnosticar la forma de la distribución (media y mediana sobre el histograma)

**Cuándo usarlo:** después de limpiar, cuando ya calculaste las medidas de la sección 22 y necesitas decidir si la media representa al caso típico. Los números solos no lo dicen — la forma sí.

📌 **Esta sección no repite las medidas.** `.mean()`, `.median()`, `.std()`, `.min()`, `.max()` y `.describe()` ya están en las secciones 2 y 22. Aquí va únicamente lo que esas secciones no cubren: **verlas dibujadas** y comparar grupos cuando la media no distingue nada.

### 36.1 Dibujar media y mediana sobre el histograma — `plt.axvline()`

La sección 26 ya grafica la distribución, pero sin puntos de referencia. `plt.axvline()` traza una línea vertical en un valor concreto: al superponer media y mediana sobre el histograma, el sesgo se ve de inmediato.

```python
import pandas as pd
import matplotlib.pyplot as plt

df = pd.read_csv('data/ventas_limpio.csv')

mean_price = df['price'].mean()      # 756
median_price = df['price'].median()  # 457

# Histograma
plt.hist(df['price'], bins=50, range=(0, 3000), color='skyblue', edgecolor='black')

# Líneas de referencia
plt.axvline(mean_price, color='red', linestyle='dashed', linewidth=2, label='Media')
plt.axvline(median_price, color='green', linestyle='dashed', linewidth=2, label='Mediana')

plt.title('Distribución de Precios con Media y Mediana')
plt.xlabel('Precio')
plt.ylabel('Cantidades')
plt.legend(loc='upper left')
plt.show()
```

> 📊 **Qué deberías ver:** histograma "Distribución de Precios con Media y Mediana" — 50 bins, línea roja (media 756) y verde (mediana 457).

⚠️ **Error de tipeo que circula mucho:** `plt.hist(df['price'], color='skyblue', edgecolor='black'))` — **un paréntesis de cierre de más**. Es `SyntaxError` si lo copias tal cual. Cuenta los paréntesis antes de ejecutar.

### 🔁 Sintaxis genérica (copiar y adaptar)

```python
# Esta plantilla sirve para cualquier columna numérica
media = df["nombre_columna"].mean()
mediana = df["nombre_columna"].median()

plt.hist(df["nombre_columna"], bins=50, range=(0, 3000), color="skyblue", edgecolor="black")

plt.axvline(media,   color="red",   linestyle="dashed", linewidth=2, label="Media")
plt.axvline(mediana, color="green", linestyle="dashed", linewidth=2, label="Mediana")

plt.title("Distribución de ...")
plt.xlabel("...")
plt.ylabel("Cantidades")
plt.legend(loc="upper left")   # necesita que cada línea tenga label=
plt.show()
```

| Parámetro de `axvline()` | Para qué sirve |
|---|---|
| 1er argumento (posicional) | El valor del eje X donde se dibuja la línea (aquí, la media o la mediana) |
| `color=` | Color de la línea — usa uno distinto por medida |
| `linestyle='dashed'` | Línea punteada, para que no se confunda con una barra |
| `linewidth=2` | Grosor — con 1 se pierde entre las barras |
| `label=` | Texto de la leyenda. **Sin `label=`, `plt.legend()` sale vacío** |

💡 **`.hist()` vs `plt.hist()`** — no son lo mismo:

- `df["col"].hist(bins=n)` (sección 26) → **método** de pandas. Rápido para explorar.
- `plt.hist(df["col"], bins=n, range=(a, b))` → **función** de pyplot. Acepta `range=` y se combina de forma natural con `axvline`. Para el gráfico del informe, usa esta.

### 36.2 ⚠️ `range=` recorta el GRÁFICO, no los DATOS

Distinción conceptual que casi nunca se aclara y que provoca conclusiones falsas:

`range=(0, 3000)` solo define **el tramo del eje X que se dibuja**. Los precios mayores a 3000 **siguen contando** en `.mean()`, `.std()`, `.max()` y en cualquier cálculo.

⚠️ **Cuánto esconde ese `range=` en tu caso:** el máximo real de `price` es **36,708** (ver la tabla de 36.7). El gráfico corta en 3,000 — **doce veces antes del máximo**. Todo lo que hay entre 3,000 y 36,708 es invisible en la imagen pero está dentro de la media de 756. Si presentas ese histograma sin decirlo, estás mostrando una distribución recortada.

> 📊 **Qué deberías ver:** "Histograma de precios sin ajuste" — el eje X llega hasta \~37,000 y todo se aplasta en una sola barra. Es la prueba visual de esta advertencia.

En el dataset de ejemplo, la media de 756 se calculó con **todos** los precios, incluidos los que el gráfico esconde. Por eso la línea roja aparece "empujada" hacia la derecha por valores que no ves en pantalla.

- ¿Quieres **enfocar la vista** en el grueso de los datos? → `range=` en el gráfico.
- ¿Quieres **excluirlos del cálculo**? → filtra el DataFrame (sección 7) y recalcula. Son cosas distintas y debes documentar cuál hiciste.

### 36.3 Leer la forma: cola larga y separación media–mediana

| Lo que ves en el histograma | Qué significa | Valor típico a reportar |
|---|---|---|
| Media ≈ mediana, forma simétrica | Sin extremos dominantes | Media |
| Media **a la derecha** de la mediana, cola larga a la derecha | Sesgo positivo: pocos valores muy altos jalan el promedio | Mediana |
| Media **a la izquierda** de la mediana, cola larga a la izquierda | Sesgo negativo | Mediana |

**Caso de ejemplo (`price`):** media 756, mediana 457. El grueso de las barras está por debajo de 800 y hay una cola larga que se estira hasta 3000+. La media quedó a la derecha del bloque principal → **sesgo positivo**. Reportar 756 como "precio típico" lo sobreestima en \~65%; el producto típico ronda los 457.

💡 **Truco de lectura:** la media siempre es arrastrada **hacia la cola**. Con solo ver de qué lado de la mediana cayó la línea roja sabes hacia dónde está el sesgo, sin calcular nada más.

📌 **Conecta con 30.3 (sentinels):** un `999` o un `-999` produce exactamente este mismo patrón — cola larga y media separada de la mediana — pero **artificial**. Antes de concluir "existen productos premium", verifica que los valores altos sean reales.

📌 **Conecta con 31.5 (evaluación pre–post):** en el histograma de `price` hay una barra anómala justo donde cae la mediana (\~470), muy por encima de sus vecinas. Ese pico es la huella visual de una **imputación con mediana**: `ventas_limpio` ya pasó por tu pipeline de la sección 35. Es la prueba gráfica de lo que dice la 31.5 — rellenar muchos huecos con un mismo valor concentra la distribución y reduce la `std`.

### 36.4 Mismo promedio, comportamiento distinto

Tres grupos de un curso de matemáticas, con la misma media aprobatoria de \~50 puntos:

```python
df_math = pd.read_csv('data/notas_clase.csv')

df_class1 = df_math[df_math['class_name'] == 'Clase 1']
df_class2 = df_math[df_math['class_name'] == 'Clase 2']
df_class3 = df_math[df_math['class_name'] == 'Clase 3']

print("Media Clase 1:", df_class1['score'].mean())
print("Media Clase 2:", df_class2['score'].mean())
print("Media Clase 3:", df_class3['score'].mean())
```

| Grupo | Media | Mín | Máx | Std | Comportamiento |
|---|---|---|---|---|---|
| Clase 1 | 49.84 | 45 | 54 | 1.8 | Muy concentrada, rango estrecho |
| Clase 2 | 50.13 | 37 | 69 | 6.6 | Dispersión media |
| Clase 3 | 50.87 | 1 | 100 | 16 | Muy dispersa, rango completo |

> 📊 **Qué deberías ver:** "Distribución de puntuaciones por clase" — las tres curvas superpuestas (Clase 1 azul estrecha, Clase 2 roja, Clase 3 verde ancha) sobre la línea del promedio 50.

**La media no distingue nada** (49.84 ≈ 50.13 ≈ 50.87). La `std` y el rango mín–máx sí: 1.8 vs 16 son mundos distintos. Si el reporte solo dijera "las tres clases promedian 50", estaría ocultando toda la información útil.

📌 **Conecta con la sección 22 ("Comparar entre grupos"):** es el mismo patrón que aparece al comparar la concentración de PM2.5 entre dos países: uno con `std` ≈ 2 y otro con `std` ≈ 220. La diferencia aquí es que **las medias coinciden**, así que la dispersión es la única señal disponible.

### 36.5 Bucle sobre una LISTA DE DATAFRAMES

En la sección 33.2 el bucle recorre **nombres de columna**. Aquí recorre **DataFrames completos** — la colección son los grupos ya filtrados:

```python
df_lista = [df_class1, df_class2, df_class3]

for df in df_lista:
    print("Media:", df['score'].mean())
    print("Mínimo:", df['score'].min())
    print("Máximo:", df['score'].max())
    print("Desviación estándar:", df['score'].std())
    print("  ")
```

⚠️ **Trampa de nombres:** la variable del bucle se llama `df` y **pisa tu `df` global**. Al terminar el loop, `df` ya no es tu DataFrame principal — es `df_class3`, y cualquier código posterior que use `df` trabajará sobre el grupo equivocado, en silencio. Usa un nombre propio:

```python
for df_grupo in df_lista:
    print("Media:", df_grupo["score"].mean())
```

### 36.6 Un gráfico por iteración

```python
df_lista = [df_class1, df_class2, df_class3]
clase_num = 1

for df in df_lista:
    plt.hist(df['score'], bins=10, color='skyblue', edgecolor='black')

    plt.title(f'Distribución de Notas: Clase {clase_num}')
    plt.xlabel('Notas')
    plt.ylabel('Cantidades')
    plt.show()

    clase_num += 1
```

> 📊 **Qué deberías ver:** los tres histogramas por separado — "Distribución de Notas: Clase 1", "Clase 2" y "Clase 3". Fíjate en que cada uno trae su propio eje X (45–54 vs 1–100): ese es exactamente el problema del que habla la advertencia del final de esta subsección.

⚠️ **La línea que lo decide todo es `plt.show()`:**

- **Dentro** del bucle → cierra la figura en cada vuelta = 3 gráficos separados.
- **Fuera** del bucle → los 3 histogramas se apilan en una sola figura superpuesta.

💡 **Versión sin contador manual.** `clase_num = 1` + `clase_num += 1` funciona, pero se rompe si cambias el orden de la lista. `zip()` empareja cada DataFrame con su nombre y elimina el contador:

```python
df_lista = [df_class1, df_class2, df_class3]
nombres  = ["Clase 1", "Clase 2", "Clase 3"]

for df_grupo, nombre in zip(df_lista, nombres):
    plt.hist(df_grupo["score"], bins=10, range=(0, 100), color="skyblue", edgecolor="black")
    plt.title(f"Distribución de Notas: {nombre}")
    plt.xlabel("Notas")
    plt.ylabel("Cantidades")
    plt.show()
```

⚠️ **Sin `range=` los gráficos NO son comparables entre sí.** Matplotlib autoescala el eje X a los datos de cada grupo: la Clase 1 (45–54) y la Clase 3 (1–100) salen igual de anchas en pantalla aunque una sea 10 veces más dispersa. Para comparar dispersión **visualmente**, fija el mismo `range=` en los tres. Si no lo fijas, la comparación válida son los números (`std`, `min`, `max`), no las imágenes.

### 36.7 Diagnóstico numérico del dataset — la línea base

**Cuándo usarlo:** como fotografía de referencia del dataset **ya limpio**, antes de analizar. Todo lo que concluyas después se compara contra esta tabla.

```python
import pandas as pd
df = pd.read_csv('data/ventas_limpio.csv')

# describe() sobre las columnas numéricas
print(df[['order_value', 'customer_age', 'price', 'quantity']].describe())
```

Salida real sobre `ventas_limpio` (5,000 filas):

| Métrica | `order_value` | `customer_age` | `price` | `quantity` |
|---|---|---|---|---|
| count | 5000 | 5000 | 5000 | 5000 |
| mean | 10,075.52 | 49.12 | 756.39 | 32.36 |
| std | 12,406.60 | 17.71 | 1,173.27 | 93.40 |
| min | 12 | 18 | 12 | 1 |
| 25% | 3,094 | 34 | 218 | 7 |
| 50% (mediana) | 10,341 | 49 | 457 | 14 |
| 75% | 13,160.50 | 64 | 847.25 | 23 |
| max | 303,824 | 80 | 36,708 | 2,083 |

✅ **Primera lectura — validación de tu pipeline:** `count` = 5000 en las cuatro columnas → **cero nulos**. La limpieza de la sección 35 funcionó. Y `quantity` tiene `min` = 1, no 0 ni negativos → la limpieza de `quantity <= 0` (sección 31.4) también quedó.

#### Dos razones derivadas que `describe()` no te da

La tabla cruda no permite comparar columnas entre sí porque están en unidades distintas (pesos vs años vs unidades). Estas dos razones sí:

| Columna | CV = `std` / `mean` | `max` / mediana | Lectura |
|---|---|---|---|
| `quantity` | 2.89 | 149× | La más inestable del dataset |
| `price` | 1.55 | 80× | Cola larga confirmada (sesgo positivo) |
| `order_value` | 1.23 | 29× | Dispersa pese a media ≈ mediana — ver abajo |
| `customer_age` | 0.36 | 1.6× | La única acotada — ver abajo |

💡 **Coeficiente de variación (CV = std / mean):** vuelve comparable la dispersión entre columnas de unidades distintas. Regla práctica: **CV \> 1 significa que la desviación estándar es mayor que la propia media** — los datos son más "ruido" que "valor típico". Tres de las cuatro columnas del dataset de ejemplo están en esa situación.

#### ⚠️ `order_value` rompe la regla de media vs mediana

Media 10,075 y mediana 10,341: **casi idénticas**. Por la heurística de la 36.3 la clasificarías como simétrica y sin problemas. Pero:

- Su `std` (12,406) es **mayor que su propia media** (CV 1.23).
- Su `max` (303,824) es **29 veces la mediana**.
- La media quedó incluso **por debajo** de la mediana, pese a esa cola enorme.

Eso solo pasa si el grueso de los pedidos está empujando hacia abajo (un bloque grande de valores bajos, posiblemente distribución **bimodal**) mientras unos pocos pedidos gigantes jalan hacia arriba, y ambos efectos se cancelan en el promedio.

🎯 **Matiz importante a la 36.3: media ≈ mediana NO garantiza ausencia de outliers.** Solo dice que los extremos están *balanceados*. Para descartar outliers hay que mirar además **`std` contra la media** y el **`max` contra la mediana**. `order_value` es tu contraejemplo dentro del propio dataset.

#### ⚠️ `customer_age` no está "sana": está uniforme

Mira los cuartiles: **18 → 34 → 49 → 64 → 80**. Los saltos son 16, 15, 15, 16 — prácticamente idénticos. Eso es la firma de una **distribución uniforme**, no de una población real de clientes.

Comprobación: la desviación estándar teórica de una uniforme entre 18 y 80 es (80 − 18) / √12 = **17.90**. La observada es **17.71**. Coinciden.

**Qué significa:** `customer_age` es dato **sintético generado al azar**. Que media ≈ mediana ahí no es señal de "columna bien comportada" — es que la columna **no tiene forma**. No esperes encontrar segmentos de edad ni patrones de comportamiento por edad en este dataset; no hay ninguno que descubrir. Documéntalo antes de invertir análisis en ella.

📌 **Conecta con 31.3:** ahí imputaste `customer_age` con la mediana (49). En una columna uniforme esa imputación crea un pico artificial justo en el centro — el mismo efecto que se ve en `price` (36.3) y que dice la 31.5.

#### ⚠️ Error de redacción muy repetido

Se lee a menudo que con `std` alta *"los valores están concentrados lejos de la media"*. Es al revés: `std` alta significa que están **dispersos**, repartidos lejos de la media. "Concentrados" describe justo lo contrario (`std` baja).

📌 **Conecta con la sección 37.4:** esta es la línea base **numérica**. La categórica (`product_category`, `payment_method`, `city`, `state`) está en la 37.4, y en la 37.6 se aplica el mismo criterio que usaste aquí con `customer_age` — distinguir qué columnas tienen señal real y cuáles son dato sintético uniforme.

### Términos clave

`plt.hist()` (función) vs `.hist()` (método) · `plt.axvline()` · `label=` + `plt.legend()` · `bins` · **`range=`** (recorte visual, no de datos) · Cola larga · **Sesgo positivo** (media \> mediana) · Distribución simétrica · Dispersión (`std`) · Rango mín–máx · Bucle sobre lista de DataFrames · Sombreado de variable (`df` del loop pisando el global) · `zip()` · f-string en `plt.title()` · **Coeficiente de variación (CV = std/mean)** · Cuartiles (25%, 50%, 75%) · Distribución uniforme · Distribución bimodal · Línea base del dataset.

📌 **Conecta con:** sección 2 y 22 (las medidas), 26 (validación visual y `.hist()`), 30.3 (sentinels que fabrican colas falsas), 31.5 (el pico de la imputación), 33.2 (bucles sobre columnas).

---

## 37. Medidas descriptivas en columnas categóricas

**Cuándo usarlo:** cuando la columna guarda **etiquetas**, no cantidades. Ahí no hay media ni desviación que calcular — la pregunta cambia de "¿cuánto?" a "¿cuáles existen, cuántas veces aparece cada una y hay alguna que domine?".

📌 **Esta sección no repite lo que ya tienes.** `.describe()` sobre texto (`count`, `unique`, `top`, `freq`) está en la **sección 2**; `value_counts()`, `.nunique()`, `.unique()` y `dropna=False` están en las secciones **2, 15, 30.1 y 30.3**. Aquí va solo lo nuevo: **frecuencia relativa**, `.mode()`, la tabla de decisión por tipo de columna y la línea base categórica del dataset.

### 37.1 Frecuencia relativa — `value_counts(normalize=True)`

`value_counts()` te da el conteo **absoluto**. Con `normalize=True` te da la **proporción** sobre el total: es lo que te deja decir "el 54% de los pedidos se pagan con tarjeta" en vez de "2,737 pedidos".

```python
import pandas as pd
df = pd.read_csv('data/ventas_limpio.csv')

# Frecuencia absoluta
print(df['product_category'].value_counts())

# Frecuencia relativa (proporción sobre el total)
print(df['product_category'].value_counts(normalize=True))
```

⚠️ **`normalize=True` devuelve PROPORCIONES, no porcentajes.** La salida es `0.1478`, no `14.78`. Si vas a ponerlo en un reporte, conviértelo tú:

```python
# Porcentaje con 2 decimales, listo para presentar
(df["nombre_columna"].value_counts(normalize=True) * 100).round(2)
```

💡 **Recordatorio de la sección 2:** `value_counts()` **ignora los `NaN` por defecto**. Si quieres que los huecos aparezcan como una categoría más, usa `dropna=False`. Y ojo: `normalize=True` calcula la proporción **sobre los valores contados**, así que con y sin `dropna=False` los porcentajes cambian.

### 37.2 `.mode()` — la moda

La moda es la categoría más frecuente. Es el equivalente categórico de "valor típico" (la media y la mediana de la sección 22 no aplican aquí).

```python
df["nombre_columna"].mode()      # devuelve una SERIE
df["nombre_columna"].mode()[0]   # el valor concreto
```

⚠️ **`.mode()` devuelve una Serie, no un valor suelto**, porque puede haber **empate**: si dos categorías tienen exactamente la misma frecuencia máxima, devuelve las dos. Por eso necesitas `[0]` para extraer un valor — y por eso vale la pena revisar `len(df["col"].mode())` antes de reportar "la categoría dominante es X".

| Quiero… | Uso | Qué devuelve |
|---|---|---|
| La categoría más frecuente | `.mode()` o el `top` de `.describe()` | La etiqueta (ej. `Fashion`) |
| Cuántas veces aparece | El `freq` de `.describe()` | El conteo (ej. 739) |
| Todas las categorías con su conteo | `.value_counts()` | Serie completa ordenada |
| Su peso relativo | `.value_counts(normalize=True)` | Proporciones que suman 1 |
| Cuántas categorías distintas hay | `.nunique()` | Un número |

⚠️ **Confusión frecuente:** se define `freq` como *"la frecuencia con la que aparece ese valor top, es decir, la moda"*. Está mal — **`top` es la moda**; `freq` es **cuántas veces aparece** la moda. Es un copia-pega del punto anterior.

### 37.3 Qué medida aplicar según el tipo de columna

| Tipo | Qué representa | Medidas principales | Pregunta que responde |
|---|---|---|---|
| **Numérica** | Cantidades sobre las que tiene sentido operar | `.mean()` `.median()` `.std()` `.min()` `.max()` `.quantile()` | ¿Cuánto? ¿Qué tan disperso? ¿Hay outliers? |
| **Categórica** | Etiquetas, grupos, estados | `.value_counts()` `.value_counts(normalize=True)` `.nunique()` `.mode()` | ¿Cuáles existen? ¿Cuántas veces? ¿Hay desbalance? |

🎯 **La distinción que importa:** un ID numérico (`order_id`, `customer_id`) **es categórico aunque sea un número**. Sacarle la media no significa nada. Esto es exactamente lo que decides en la **sección 30.1** al clasificar por rol y no por `dtype`.

### 37.4 Línea base categórica del dataset

El equivalente categórico de la tabla numérica de la 36.7:

```python
columnas_categoricas = ['product_category', 'payment_method', 'city', 'state']
print(df[columnas_categoricas].describe())
```

| Columna | count | unique | top | freq | % del top |
|---|---|---|---|---|---|
| `product_category` | 5000 | 8 | Fashion | 739 | 14.8% |
| `payment_method` | 5000 | 4 | credit_card | 2,737 | 54.7% |
| `city` | 5000 | 11 | Houston | 513 | 10.3% |
| `state` | 5000 | 10 | CA | 977 | 19.5% |

`value_counts()` completo de `product_category`:

```text
Fashion        739      0.1478
Electronics    735      0.1470
Beauty         721      0.1442
Toys           715      0.1430
Sports         702      0.1404
Grocery        684      0.1368
Home           679      0.1358
?               25      0.0050
```

### 37.5 ⚠️ Cuando un sentinel sobrevive a la limpieza

En la salida de arriba, sobre el archivo ya "limpio", aparece **`?` con 25 registros (0.50%)**.

No debería estar ahí. La **sección 30.3** identificó `"?"` como sentinel de texto de `product_category`, y la **sección 35.2** (`reemplazar_sentinels_global`) existe precisamente para convertirlo en `NaN`. Hay dos explicaciones posibles y hay que distinguirlas:

1. El archivo que estás leyendo **no pasó por tu pipeline** — es una versión "limpia" de otro origen, no la tuya.
2. Tu pipeline sí corrió pero **no cubrió esta columna** (la lista `columnas_texto` o el valor buscado no coincidieron).

**Cómo verificarlo:**

```python
df["product_category"].value_counts(dropna=False)   # ¿sigue el "?" y hay NaN?
df["product_category"].isin(["?"]).sum()            # cuántos exactamente
```

⚠️ **Consecuencia inmediata:** `.describe()` reporta `unique = 8` para `product_category`. Son **7 categorías reales + 1 sentinel**. `unique` cuenta basura como si fuera categoría, y es facilísimo reportar "product_category tiene 8 categorías" sin notarlo. **Nunca reportes `unique` sin haber revisado antes `value_counts()`.**

📌 **A verificar también:** en la **sección 35.4** imputaste `city` y `state` con `"unknown"`. Si eso corrió sobre este DataFrame, `"unknown"` es **una de las 11 ciudades y uno de los 10 estados** que reporta la tabla de arriba — serían 10 ciudades reales y 9 estados reales. Confírmalo con `df["city"].value_counts()`. Un valor imputado por ti sigue apareciendo como categoría; no es un error, pero sí algo que debes descontar al reportar cardinalidad.

### 37.6 ⚠️ "La categoría más frecuente" no siempre es un hallazgo

`.describe()` **siempre** devuelve un `top`, incluso cuando todas las categorías son prácticamente iguales. Que exista un máximo no significa que haya dominancia.

**Caso `product_category`:** 4,975 registros válidos repartidos en 7 categorías. Si fueran equiprobables, lo esperado sería 4,975 / 7 = **710.7** por categoría, con una desviación típica de √(4975 × 1/7 × 6/7) ≈ **24.7**.

- Fashion (739) está a **+1.15 desviaciones** de lo esperado.
- Home (679) está a **−1.28 desviaciones**.

Todo el rango observado (60 registros de diferencia entre la primera y la última) es **exactamente lo que produce el azar**. `product_category` está uniformemente repartida: **"Fashion es la categoría dominante" es ruido, no un hallazgo.**

| Columna | Esperado si fuera uniforme | Observado en el top | ¿Dominancia real? |
|---|---|---|---|
| `payment_method` | 1,250 (25%) | credit_card 2,737 (54.7%) | **Sí** — más del doble de lo esperado |
| `state` | 500 (10%) | CA 977 (19.5%) | **Sí** — casi el doble |
| `city` | \~455–490 | Houston 513 (10.3%) | **Dudoso** — depende de si `unknown` está entre las 11 |
| `product_category` | 710.7 ± 24.7 | Fashion 739 (+1.15σ) | **No** — dentro del ruido |

💡 **Regla práctica antes de escribir "X domina":** compara la frecuencia observada contra `total / nº de categorías`. Si la diferencia es de unos pocos puntos porcentuales, no lo reportes como patrón. Es tentador escribir *"Houston, Seattle y Los Angeles representan más del 30% de los clientes"* — pero con 11 ciudades casi iguales, **cualesquiera 3 suman \~27%**. Eso no es un hallazgo, es aritmética.

📌 **Conecta con 36.7:** es el mismo fenómeno que aparece en `customer_age` (uniforme entre 18 y 80). Buena parte de este dataset de ejemplo es **dato sintético generado al azar**; las columnas con señal real son `payment_method`, `state`, `price`, `quantity` y `order_value`. Documenta cuáles son cuáles antes de invertir análisis en las que no tienen nada que revelar.

### Términos clave

`value_counts(normalize=True)` (frecuencia relativa, devuelve **proporciones**) · `dropna=False` · `.mode()` (devuelve **Serie**, puede haber empate) · `top` (la moda) vs `freq` (su conteo) · Cardinalidad (`unique`, `.nunique()`) · Frecuencia absoluta vs relativa · Categoría dominante vs **desbalance dentro del ruido** · ID numérico ≠ variable numérica.

📌 **Conecta con:** sección 2 (`.describe()` según tipo), 15 (`.nunique()`), 30.1 (clasificar por rol, no por dtype), 30.3 (sentinels `"?"`), 35.2 y 35.4 (pipeline y `"unknown"`), 36.7 (línea base numérica).

---

## 38. Construir e interpretar histogramas (bins, bordes y escala)

**Cuándo usarlo:** cuando ya sabes **qué** quieres ver (la forma de la distribución, sección 36) y ahora tienes que **construir** el gráfico bien. Un histograma mal configurado no es un gráfico feo: es un gráfico que dice algo distinto de lo que pasa en los datos.

📌 **Esta sección no repite lo básico.** `plt.hist()`, `bins`, `color`, `edgecolor`, etiquetas y `plt.show()` están en las secciones 26 y 36.1; `range=` y su matiz en 36.2; sesgo y cola larga en 36.3. Aquí va lo nuevo: **cómo elegir el número de bins**, **leer los bordes reales** de cada barra, `kde` en Seaborn y **cómo decidir dónde cortar el eje X**.

### 38.1 Qué muestra un histograma — y qué decisiones esconde

Divide una columna numérica en rangos y cuenta cuántos valores caen en cada uno.

- **Eje X:** los rangos (20–29, 30–39, …).
- **Eje Y:** cuántas observaciones hay en cada rango.

> 📊 **Qué deberías ver:** histograma de ejemplo "Distribución de las edades" (barras de 20 a 70).

Lo que puedes identificar de un vistazo:

| Lo que ves | Qué significa |
|---|---|
| Barras agrupadas en pocos rangos | Concentración — `std` baja |
| Barras repartidas por todo el eje | Dispersión — `std` alta |
| Cola que se estira a un lado | Sesgo (ver 36.3 para el lado y qué reportar) |
| Varios picos separados | **Multimodalidad** — hay subgrupos distintos mezclados |
| Barras minúsculas muy lejos del bloque | Valores extremos / posibles outliers |

🎯 **Histograma ≠ gráfico de barras.** El histograma es para variables **numéricas**: tú decides en cuántos rangos se agrupan y las barras van pegadas porque el eje es continuo. El gráfico de barras / `countplot` es para **categóricas**: una barra por categoría, sin decisión de por medio (secciones 26 y 37). Confundirlos es reportar mal el tipo de variable.

### 38.2 Matplotlib vs Seaborn — el parámetro `kde`

```python
# Matplotlib — control total, aspecto básico
plt.hist(df['score'], bins=10, color='skyblue', edgecolor='black')
plt.xlabel('Notas de los estudiantes')
plt.ylabel('Cantidad de estudiantes')
plt.title('Distribución de notas')
plt.show()
```

```python
# Seaborn — más estético, permite la curva de densidad
import seaborn as sns

sns.histplot(df['score'], bins=10, color='skyblue', kde=True)
plt.xlabel('Notas de los estudiantes')
plt.ylabel('Cantidad de estudiantes')
plt.title('Distribución de notas')
plt.show()
```

|   | `plt.hist()` | `sns.histplot()` |
|---|---|---|
| Aspecto por defecto | Básico | Más pulido |
| Curva de densidad | No | `kde=True` |
| Devuelve `counts` y `bin_edges` | **Sí** (ver 38.4) | No directamente |
| Acepta `range=` | **Sí** | No (se usa `binrange=`) |

💡 **Qué es `kde`:** una curva suavizada que estima la forma general de la distribución. Sirve para **comunicar** la tendencia en una presentación. ⚠️ Pero es una **estimación**, no tus datos: con pocos registros puede inventar curvas suaves donde en realidad hay huecos, y siempre extiende la curva más allá del mínimo y el máximo reales. **Decide con las barras, presenta con la curva.**

⚠️ **Inconsistencia típica:** el código usa `kde=True` pero la viñeta de abajo describe `kde=False`. Son los dos casos, no el mismo.

### 38.3 Elegir el número de bins

Es la decisión que más cambia el gráfico — y los datos no cambian nada.

- **Pocos bins** → todo se ve como una sola montaña; esconde patrones reales.
- **Demasiados bins** → aparecen "dientes" y ruido; cuesta ver la forma.

> 📊 **Qué deberías ver:** los tres histogramas de `math_class` — 5 bins, 10 bins y 30 bins.

**Procedimiento práctico:**

1. Empieza con `bins=10`.
2. Si se ve grueso y sin detalle → sube a 12, 15 o más.
3. Si se ve ruidoso y dentado → baja a 8 o menos.

💡 **Regla de la raíz cuadrada:** `bins ≈ √n`, donde `n` es el número de registros.

```python
import numpy as np
bins_sugeridos = int(np.sqrt(len(df)))   # 900 registros → 30 bins
```

⚠️ **La regla y la recomendación habitual se contradicen.** `math_class` tiene 300 estudiantes: √300 ≈ **17 bins**, pero la recomendación habitual es 10, y con 30 se dice que ya es ruido. Las fórmulas son **punto de partida**, no resultado — el ajuste final es visual.

🎯 **El número de bins no cambia los datos, cambia la resolución con la que los miras.** No existe un `bins` "correcto": existe el que revela la forma sin inventar detalle. Por eso, en un reporte, **di con cuántos bins graficaste** — es una decisión tuya, no un hecho del dataset.

### 38.4 Ver los bordes reales de cada barra — `bin_edges` + `xticks()`

`plt.hist()` **devuelve tres cosas**. Normalmente las ignoramos, pero capturarlas te dice exactamente dónde empieza y termina cada barra:

```python
counts, bin_edges, _ = plt.hist(df_math['score'], bins=10, color='skyblue', edgecolor='black')
plt.xticks(bin_edges)   # usa los bordes reales como etiquetas del eje X
plt.show()
```

| Valor devuelto | Qué contiene |
|---|---|
| `counts` | Array con **cuántos datos** cayeron en cada barra |
| `bin_edges` | Array con los **bordes** de las barras — el que interesa |
| `_` (tercero) | El **`BarContainer`**: los objetos gráficos de las barras. Se descarta con `_` |

⚠️ **Definición equivocada muy repetida:** se dice que el `_` son "los demás valores". No son valores — son los objetos de dibujo de matplotlib. El guion bajo es la convención de Python para "esto lo recibo pero no lo voy a usar".

💡 **`bin_edges` tiene un elemento MÁS que `bins`.** Con `bins=10` obtienes **11** bordes: 10 barras necesitan 11 fronteras. Si lo usas para construir etiquetas, cuenta con ese +1.

⚠️ **Elige límites que den bordes redondos.** Con `range=(12, 1000)` y `bins=10`, cada barra mide (1000 − 12) / 10 = **98.8**, así que `xticks(bin_edges)` te llena el eje de `12, 110.8, 209.6, 308.4…` — ilegible. Ajusta para que la división sea limpia:

```python
# 10 barras de 100 exactos: bordes en 0, 100, 200, ... 1000
plt.hist(df["price"], bins=10, range=(0, 1000), color="skyblue", edgecolor="black")
```

### 38.5 Decidir dónde cortar el eje X — usa el 75% de `describe()`

Cuando hay outliers, el eje se estira para incluirlos: las barras se aplastan y **parece** que todos los datos están en un solo bloque.

> 📊 **Qué deberías ver:** "Histograma de precios sin ajuste" (eje hasta \~37,000, una sola barra visible).

**Procedimiento:**

```python
df['price'].describe()   # mira el 75% (tercer cuartil)
```

El **75%** significa "tres de cada cuatro valores están por debajo de este número". Es tu punto de referencia: corta **un poco por encima** de él.

| Si el 75% es… | Corta el eje en… |
|---|---|
| 420 | 500 o 600 |
| 847 (caso `price` del ejemplo) | 1,000 |
| 1,200 | 1,500 |

🎯 **El cuartil que uses depende del LADO del sesgo:**

- **Cola larga a la derecha** (el caso habitual: precios, ingresos, gastos) → el problema está arriba. Límite inferior = el **mínimo** de `describe()`; límite superior = un poco por encima del **75% (Q3)**.
- **Cola larga a la izquierda** (ratings, calificaciones, descuentos) → el problema está abajo. Límite inferior = un poco por **debajo del 25% (Q1)**; límite superior = el **máximo** de `describe()`.

```python
# Sesgo a la derecha
plt.hist(df["col"], bins=10, range=(minimo, un_poco_arriba_de_Q3))

# Sesgo a la izquierda
plt.hist(df["col"], bins=10, range=(un_poco_abajo_de_Q1, maximo))
```

💡 Para saber de qué lado está la cola antes de graficar, compara media y mediana (36.3): **media \> mediana** → cola a la derecha; **media \< mediana** → cola a la izquierda.

```python
counts, bin_edges, _ = plt.hist(df['price'], bins=10, range=(0, 1000),
                                color='skyblue', edgecolor='black')
plt.xticks(bin_edges)
plt.title('Histograma de precios con ajuste de eje X')
plt.show()
```

> 📊 **Qué deberías ver:** "Histograma de precios con ajuste de eje X" (las dos versiones: con ticks por defecto y con `xticks(bin_edges)`).

📌 **Conecta con 36.2:** `range=` **no elimina datos**. La media de 756, la `std` y el `max` de 36,708 siguen calculándose con todos los precios; solo cambiaste la ventana por la que miras. Documenta siempre que el gráfico está recortado.

### 38.6 ⚠️ Leer el resultado sin sobreinterpretar

Del histograma ajustado de `price`, la conclusión tentadora es: *"la mayoría de precios están concentrados entre 400 y 500 dólares"*. Dos problemas:

**1. No es "la mayoría".** Esa barra tiene 775 registros de los \~3,956 visibles en el rango 0–1,000 → **19.6%**. Sobre las 5,000 filas del dataset, un **15.5%**. Es el **bin modal**, que es otra cosa: la barra más alta, no la mayoría de los datos.

**2. Ese pico probablemente no es real.** La mediana de `price` es **457** — justo dentro de esa barra. Es la **huella de la imputación con mediana** que aparece en 36.3 y cuyo efecto describe la 31.5. Es fácil presentarlo como "los picos reales" de la distribución.

🎯 **Regla antes de llamar "pico" a una barra:** compara el centro de esa barra con la **mediana** y con los **valores que tú usaste para imputar**. Si coinciden, el pico lo creaste tú al limpiar, no está en el negocio. Y usa "la mayoría" solo cuando la suma pase del 50% — si no, di "el rango más frecuente".

### Términos clave

`bins` (resolución, no dato) · Regla de la raíz cuadrada (`√n`) · `counts` · **`bin_edges`** (siempre `bins + 1` elementos) · `BarContainer` / `_` · `plt.xticks()` · `kde=True` (estimación suavizada, no los datos) · `sns.histplot` vs `plt.hist` · Tercer cuartil (75%) como referencia para cortar el eje · **Bin modal ≠ mayoría** · Multimodalidad · Pico de imputación.

📌 **Conecta con:** sección 26 (validación visual), 31.5 (efecto de imputar), 36.1 (media y mediana sobre el histograma), 36.2 (`range=` recorta el gráfico), 36.3 (leer el sesgo), 36.7 (el 75% real de cada columna), 37 (por qué las categóricas no van con histograma).

---

## 40. Interpretar la forma de una distribución en clave de negocio

**Cuándo usarlo:** al final del flujo. Ya limpiaste (35), mediste (36 y 37) y graficaste (38 y 39). Aquí traduces la forma a **una frase que alguien pueda usar para decidir** — y, sobre todo, verificas que esa frase se sostenga.

### 40.1 Las tres formas y la historia que cuenta cada una

> 📊 **Qué deberías ver:** las tres curvas de referencia (`40-1_curva_normal.png`, `40-1_sesgo_derecha.png`, `40-1_sesgo_izquierda.png`).

| Forma | Señal numérica | Ejemplos en retail | Qué reportar | Lectura de negocio |
|---|---|---|---|---|
| **Normal / simétrica** | media ≈ mediana ≈ moda | Tiempos de entrega estables, edades en mercados amplios | La **media** describe bien | Comportamiento estable y predecible; el proceso está bajo control |
| **Sesgo a la derecha** (right-skewed) | **media \> mediana**; cola larga arriba | Gasto de clientes, precios, ingresos, salarios | La **mediana** como valor típico; la media sobreestima | Un grupo pequeño mueve una parte grande del negocio (*long-tail*). Identificar clientes de alto valor es la acción |
| **Sesgo a la izquierda** (left-skewed) | **media \< mediana**; cola larga abajo | Ratings y reseñas, satisfacción, descuentos | La **mediana**; el promedio castiga de más | La mayoría está bien y hay casos negativos aislados. La acción es investigar las excepciones, no al cliente promedio |

🎯 **La forma no es el hallazgo: la forma es la pregunta.** "Está sesgada a la derecha" no es un insight, es una descripción. El insight es *"el 20% superior de pedidos genera el X% de los ingresos, así que conviene un programa de retención para ese grupo"* — y eso requiere calcularlo, no mirarlo.

### 40.2 Las tres columnas del dataset de ejemplo

```python
import seaborn as sns
import matplotlib.pyplot as plt

sns.histplot(df['order_value'], bins=50, color='skyblue', kde=True)
plt.show()
```

> 📊 **Qué deberías ver:** `40-2_order_value_kde.png`, `40-2_customer_age_kde.png` y `40-2_price_kde.png`.

| Columna | Lo que se ve | Lectura |
|---|---|---|
| `order_value` | Masa concentrada abajo, cola hasta 300,000. **Y dos picos**, no uno | Sesgo a la derecha **con bimodalidad**: hay tickets pequeños y un segundo grupo de tickets medianos. Candidatos a "whales" arriba — valida si son compras corporativas antes de llamarlos clientes premium |
| `price` | Sesgo a la derecha, menos extremo | Catálogo mayormente accesible con una franja premium. Mezcla plausible de surtido |
| `customer_age` | Plano de 18 a 80 con **un solo pico** en 49–55 | ⚠️ Ver 40.3 — ese pico no es un segmento |

📌 **La bimodalidad de `order_value` confirma lo que ya se deducía en 36.7** sin ver el gráfico: la media (10,075) quedó **por debajo** de la mediana (10,341) pese a un máximo de 303,824. Dos poblaciones mezcladas explican exactamente eso. Etiquetarla solo como "right-skewed" pierde lo interesante: **hay dos tipos de pedido**, y separarlos vale más que describir la cola.

### 40.3 ⚠️ El caso `customer_age` — cuando el "hallazgo" lo creaste tú al limpiar

Conteos por barra del histograma con `bins=10`:

```text
500 · 460 · 458 · 470 · 495 · 675 · 463 · 490 · 447 · 525
                              ↑
```

Todo ronda 480. **Una sola barra se dispara a 675.**

**La cuenta:** el rango es 18–80, así que con 10 bins cada barra mide (80 − 18) / 10 = **6.2 años**. Los bordes caen en 18, 24.2, 30.4, 36.6, 42.8, **49**, 55.2… La barra del pico es exactamente **\[49, 55.2)**.

**El dato que lo explica:** la mediana de `customer_age` es **49** (tabla de 36.7). Y en la **sección 31.3** imputaste esa columna **con la mediana** — alrededor de 150 registros, \~3%. El exceso de esa barra sobre sus vecinas es ≈195.

**Conclusión: el pico es tu imputación.** No existe un segmento de clientes de 50 a 55 años.

⚠️ **La conclusión tentadora — y equivocada:** *"el corazón del negocio está en los adultos maduros"*, *"un segmento destacado entre 50 y 55 años"*, *"útil para campañas diferenciadas por edad"*. Sería una **recomendación de negocio construida sobre valores rellenados**. En un proyecto real, eso dirige presupuesto de marketing a un artefacto de limpieza.

⚠️ **La curva `kde` agrava el problema:** suaviza el pico puntual en una colina que abarca de \~42 a \~58 años, convirtiendo un artefacto de una sola barra en lo que parece una tendencia natural. Es literalmente la advertencia de tu **38.2** — *decide con las barras, presenta con la curva*.

📌 **Y la columna ya estaba marcada como sospechosa dos veces:**

- **36.7:** cuartiles equiespaciados (18 → 34 → 49 → 64 → 80) y `std` observada 17.71 vs 17.90 teórica de una uniforme. Es dato sintético.
- **39.3:** valla superior 109 contra un máximo de 80 → **cero outliers posibles**. Una uniforme no tiene cola.

Una columna uniforme **no tiene forma que interpretar**. El único relieve del gráfico lo pusiste tú.

### 40.4 Checklist antes de convertir una forma en recomendación

Recórrela **siempre** antes de escribir una conclusión de negocio a partir de un gráfico:

| # | Pregunta | Si la respuesta es sí… | Dónde |
|---|---|---|---|
| 1 | ¿El pico coincide con la **mediana** o con un valor que usé para imputar? | El pico lo creaste tú. No es un hallazgo | 31.3, 36.3, 40.3 |
| 2 | ¿El pico coincide con un **sentinel** (`999`, `0`, `"?"`)? | Es basura sin limpiar, no comportamiento | 30.3, 37.5 |
| 3 | ¿La "categoría dominante" supera lo esperado **por azar**? | Si no lo supera, es ruido: no lo reportes | 37.6 |
| 4 | ¿La forma **sobrevive** si cambio el número de bins? | Si desaparece con otro `bins`, era resolución, no patrón | 38.3 |
| 5 | ¿El gráfico está **recortado** con `range=`? | Dilo explícitamente; hay datos fuera de la imagen | 36.2, 38.5 |
| 6 | ¿Estoy leyendo la curva `kde` en vez de las barras? | La curva suaviza e inventa; verifica en el histograma | 38.2 |
| 7 | ¿**Media vs mediana** confirma lo que creo ver? | Si no coinciden, tu lectura visual está mal | 36.3 |
| 8 | ¿Dije "la mayoría" sin que pase del 50%? | Usa "el rango más frecuente"; el bin modal no es la mayoría | 38.6 |

🎯 **El criterio de fondo:** antes de preguntarte *qué significa esta forma para el negocio*, pregúntate *de dónde salió esta forma*. Casi siempre hay tres candidatos antes que el comportamiento real de los clientes: **tu limpieza**, **la configuración del gráfico** y **el azar**.

### Términos clave

Distribución normal / simétrica · **Right-skewed** (media \> mediana) · **Left-skewed** (media \< mediana) · Cola larga · *Long-tail effect* · **Whales** (clientes de alto valor) · Bimodalidad · Segmento dominante · **Pico de imputación** (artefacto, no hallazgo) · Artefacto de limpieza vs comportamiento real · Insight ≠ descripción de la forma.

📌 **Conecta con:** sección 31.3 y 31.5 (la imputación que creó el pico), 35 (el pipeline), 36.3 y 36.7 (media vs mediana, línea base numérica), 37.6 (dominancia real vs ruido), 38.2, 38.3 y 38.6 (kde, bins y sobreinterpretación), 39.3 (por qué `customer_age` no puede tener outliers).

---

## 45. Comparar una distribución entre grupos (`hue` en histplot)

**Cuándo usarlo:** cuando quieres ver si dos grupos (plan, país, segmento) se comportan distinto en una misma variable numérica. Es el histograma de la sección 38, partido por una categórica.

### ⚠️ La trampa: conteos vs. proporciones

Si los grupos tienen **tamaños distintos**, el histograma por conteo miente. El grupo más numeroso siempre tendrá barras más altas — sin que eso signifique que usa más.

Ejemplo real (ConnectaTel): 64.9% de los clientes son Básico y 35.1% Premium. Un histograma por conteo hace que Básico se vea "más" en todos los tramos, cuando solo hay más clientes Básico.

Es el mismo principio de la sección 37.1 (`value_counts(normalize=True)`): **cuando comparas grupos de tamaño distinto, las proporciones dicen la verdad y los conteos engañan.**

### 🔁 Sintaxis genérica (copiar y adaptar)

```python
plt.figure(figsize=(10, 5))
sns.histplot(
    data=df,
    x="columna_numerica",
    hue="columna_grupo",
    palette=['skyblue', 'green'],
    stat="probability",     # eje Y en proporción, no en conteo
    common_norm=False,      # cada grupo normaliza por SEPARADO
    multiple="dodge",       # barras lado a lado
    bins=20                 # o discrete=True si son enteros
)
plt.title("Distribución de X por grupo")
plt.xlabel("Etiqueta de X")
plt.ylabel("Proporción dentro de cada grupo")
plt.show()
```

### Los parámetros que importan

| Parámetro | Qué hace | Por qué |
|---|---|---|
| `hue="col"` | Parte el histograma por categoría | Es lo que permite la comparación |
| `stat="probability"` | Eje Y en proporción (0–1) en vez de conteo | Quita el efecto del tamaño del grupo |
| `common_norm=False` | **Cada grupo suma 1 por su cuenta** | Sin esto la normalización es sobre el total y el sesgo vuelve |
| `multiple="dodge"` | Barras lado a lado | Por defecto se encima (`layer`) y con 2+ grupos se lee mal |
| `discrete=True` | Una barra por valor entero | Para conteos con pocos valores (0–17); sustituye a `bins` |
| `bins=n` | Número de barras | Solo para variables continuas |

### ⚠️ Nota de versión de seaborn

- `stat="percent"` (eje 0–100) **requiere seaborn ≥ 0.12**.
- En **0.11** usa `stat="probability"` (eje 0–1). Hace lo mismo, cambia la escala.
- Comprobar con: `print(sns.__version__)`

`common_norm` y `discrete` sí existen desde 0.11.

### Cómo leer el resultado

- **Distribuciones que se superponen casi por completo** → el grupo NO explica la variable. Es un hallazgo, no un fracaso: significa que la categoría contratada no refleja el comportamiento real.
- **Distribuciones desplazadas** → un grupo usa sistemáticamente más o menos.
- **Misma media pero distinta dispersión** → mismo promedio, comportamiento distinto (ver 36.4).

### Términos clave

- **Normalización por grupo** — dividir cada grupo entre su propio total para que sumen 1 cada uno.
- **`common_norm`** — decide si la normalización se hace sobre el total (`True`) o por grupo (`False`).

---

[← Python aplicado a la limpieza](07-python-para-limpieza.md) · [Índice](../README.md) · [Outliers: detección y tratamiento →](09-outliers.md)
