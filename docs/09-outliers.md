# Outliers: detección y tratamiento

[← Índice de la guía](../README.md)

---

## 39. Boxplots y detección de outliers (IQR)

**Cuándo usarlo:** cuando el histograma ya te mostró la forma y necesitas responder lo que el histograma no responde bien: **¿dónde están los outliers y qué tan lejos están del resto?**

📌 **Esta sección no repite lo básico.** La anatomía del boxplot (caja Q1–Q3, mediana, bigotes, puntos) está en la **sección 26**, y los pasos básicos son literalmente las secciones **38.4 y 38.5** con `order_value` en vez de `price`. Aquí va lo nuevo: el **IQR** como medida, **de dónde salen los bigotes**, la lectura comparada histograma/boxplot y las vallas calculadas sobre el dataset de ejemplo.

### 39.1 El IQR — dispersión robusta

**IQR (rango intercuartil) = Q3 − Q1.** Es el ancho de la caja: mide qué tan disperso está el **50% central** de los datos.

```python
q1  = df["nombre_columna"].quantile(0.25)
q3  = df["nombre_columna"].quantile(0.75)
iqr = q3 - q1
```

🎯 **IQR vs `std` — por qué tener las dos:**

|   | `std` | IQR |
|---|---|---|
| Qué usa | **Todos** los valores | Solo Q1 y Q3 |
| Ante un outlier | Se dispara | No se mueve |
| Sirve para | Medir la dispersión real, incluidos extremos | Medir la dispersión **del grueso** de los datos |

📌 **Conecta con 36.7:** ahí viste que `order_value` tiene una `std` (12,406) **mayor que su propia media**. Su IQR es 10,066 — la caja también es ancha, así que en ese caso la dispersión es real y no solo cosa de unos pocos extremos. Comparar `std` con IQR te dice si la dispersión viene del grueso o de la cola.

### 39.2 De dónde salen los bigotes — la regla del 1.5 × IQR

Esto es lo que casi nunca se explica, y sin ello "outlier" queda como algo que se reconoce a ojo. Las fronteras se llaman **vallas** (*fences*):

```text
valla inferior = Q1 − 1.5 × IQR
valla superior = Q3 + 1.5 × IQR
```

Todo dato fuera de esas vallas se dibuja como **punto suelto**: eso es un outlier según el boxplot.

```python
valla_inf = q1 - 1.5 * iqr
valla_sup = q3 + 1.5 * iqr

outliers = df[(df["nombre_columna"] < valla_inf) | (df["nombre_columna"] > valla_sup)]
print(len(outliers), "outliers —", round(len(outliers) / len(df) * 100, 2), "%")
```

> 📊 **Qué deberías ver:** el diagrama de anatomía del boxplot (`39_anatomia_boxplot.png`).

⚠️ **Tres errores frecuentes en los diagramas de anatomía del boxplot que circulan por ahí:**

1. **Etiqueta el extremo del bigote como "Q4". Es falso.** El bigote **no llega hasta la valla**: llega hasta el **último dato real** que cae dentro de ella. Q4 sería el máximo, y el máximo normalmente queda *fuera* del bigote, dibujado como punto.
2. **Distingue "atípico leve" (●) de "atípico extremo" (✱).** Esa separación existe en estadística (leve = más allá de 1.5·IQR, extremo = más allá de **3·IQR**), pero `sns.boxplot` y `plt.boxplot` **no la dibujan**: pintan un solo tipo de punto. Ese diagrama es estilo SPSS, no lo que produce tu código.
3. Nunca menciona el 1.5, que es justo lo que define todo el gráfico.

📌 **Conecta con la sección 41:** ahí está el **segundo método** para detectar outliers — el **Z-score** (`|z| > 3`) — y el criterio para elegir entre los dos: IQR para distribuciones sesgadas, z-score para las aproximadamente simétricas.

💡 **Por qué 1.5:** es una convención (Tukey), no una ley. En una distribución normal deja fuera ≈0.7% de los datos. Con distribuciones muy sesgadas marca **muchos** puntos como outliers sin que ninguno sea un error — por eso el boxplot **señala candidatos**, no culpables.

### 39.3 Las vallas del dataset de ejemplo

Calculadas con los cuartiles de la tabla de 36.7:

| Columna | Q1 | Q3 | IQR | Valla superior | Máximo real | Lectura |
|---|---|---|---|---|---|---|
| `customer_age` | 34 | 64 | 30 | 109 | 80 | **Cero outliers** — el máximo ni se acerca |
| `price` | 218 | 847.25 | 629.25 | 1,791 | 36,708 | Máximo **20× la valla** |
| `quantity` | 7 | 23 | 16 | 47 | 2,083 | Máximo **44× la valla** |
| `order_value` | 3,094 | 13,160.5 | 10,066.5 | 28,260 | 303,824 | Máximo **11× la valla** |

💡 **La valla inferior sale negativa en las cuatro** (ej. `price`: 218 − 943.9 = −725.9). Como ninguna de estas columnas admite valores negativos, **no puede haber outliers bajos**: todos los puntos del boxplot estarán a la derecha. Eso ya te confirma sesgo positivo sin mirar el gráfico.

📌 **`customer_age` sin un solo outlier** es otra confirmación de lo que dice la 36.7: es una columna **uniforme sintética**. Una distribución uniforme no puede generar valores atípicos, porque no tiene cola.

### 39.4 El boxplot en código

```python
import seaborn as sns
import matplotlib.pyplot as plt

sns.boxplot(data=df, x='order_value', color='skyblue')
plt.title('Boxplot de order_value')
plt.xlabel('Order value')
plt.show()
```

💡 **Dos formas de llamarlo** (las verás usadas indistintamente, casi nunca explicadas):

- `sns.boxplot(data=df, x="columna")` → pasas el DataFrame y el **nombre** de la columna.
- `sns.boxplot(x=df["columna"])` → pasas la **Serie** directamente.

Hacen lo mismo. La primera es la que escala mejor cuando añades `y=` para comparar por categoría (sección 26).

> 📊 **Qué deberías ver:** boxplot de préstamos (`39_boxplot_prestamos.png`).

**Cómo se lee ese ejemplo:** la caja es estrecha y está pegada a la izquierda → la mayoría de los préstamos son montos pequeños y muy parecidos entre sí. El bigote llega a \~10K → ese es el rango "normal". Los puntos aislados entre 40K y 80K están fuera de la valla → préstamos excepcionalmente grandes. Ojo con el hueco: **no hay nada entre 10K y 40K**, y un salto vacío así suele indicar dos poblaciones distintas mezcladas (préstamos personales vs. algo más), no una cola continua.

### 39.5 Leer sesgo, dispersión y outliers en cada gráfico

| Qué buscas | En el histograma | En el boxplot |
|---|---|---|
| **Sesgo** | De qué lado se estira la cola | Un bigote más largo que el otro; la mediana descentrada dentro de la caja; puntos solo en un extremo |
| **Dispersión** | Aplanada = alta; picuda = baja | Caja ancha = IQR grande = alta; caja estrecha = baja |
| **Outliers** | Barras solitarias muy alejadas (fáciles de perder si son de altura 1) | **Puntos individuales fuera del bigote** — la forma más clara |
| **Multimodalidad** | **Varios picos** — se ve bien | **No se ve** — el boxplot la esconde por completo |

🎯 **Por eso se usan juntos, no uno u otro.** El boxplot gana en outliers y en comparar grupos; el histograma gana en forma y es el **único** que revela si hay dos o tres poblaciones mezcladas. Un boxplot puede verse perfectamente normal sobre datos bimodales.

### 39.6 Un outlier no se borra

📌 **Retoma la sección 14 y la sección 30.3.** El boxplot **marca candidatos**; la decisión es tuya y va documentada:

| Pregunta | Si la respuesta es sí… |
|---|---|
| ¿Es físicamente imposible? (edad 200, cantidad negativa) | Error de captura → `NaN` y trátalo como ausente (secciones 30.3 y 31.4) |
| ¿Es un sentinel disfrazado? (`999`, `-999`) | No es outlier, es basura → sección 30.3 |
| ¿Es raro pero plausible? (un pedido corporativo de 300,000) | **Se queda.** Documenta que existe y reporta con mediana además de media |
| ¿Distorsiona la conclusión? | Analiza con y sin él, y **reporta las dos cifras** (patrón de la sección 31.5) |

⚠️ **Nunca elimines filas solo porque el boxplot las pintó como puntos.** Con una distribución sesgada, la regla del 1.5 marca como atípico un porcentaje alto de datos perfectamente reales — en `price`, todo lo que pase de 1,791 pesos, que incluye productos de gama alta legítimos.

### Términos clave

**IQR / rango intercuartil** (Q3 − Q1) · Cuartiles Q1, Q2, Q3 · **Vallas** (`Q1 − 1.5·IQR`, `Q3 + 1.5·IQR`) · Regla de Tukey (1.5 leve / 3 extremo) · Bigote (llega al último dato **dentro** de la valla, no a la valla) · `.quantile()` · `sns.boxplot(data=, x=)` vs `sns.boxplot(x=Serie)` · Dispersión robusta (IQR) vs no robusta (`std`) · Candidato a outlier ≠ error.

📌 **Conecta con:** sección 14 (filtrar valores imposibles), 26 (anatomía básica del boxplot), 30.3 (sentinels), 31.4 y 31.5 (limpiar y evaluar pre–post), 36.7 (los cuartiles de donde salen estas vallas), 38.5 (el flujo histograma → `describe()` → `range=`).

---

## 41. Detectar outliers con reglas estadísticas — IQR y Z-score

**Cuándo usarlo:** cuando necesitas un **criterio objetivo** para decir "esto es atípico", en vez de señalarlo a ojo en un gráfico.

📌 **El IQR ya está completo en la sección 39:** qué es, `.quantile()`, la regla del 1.5, las vallas y la tabla con los límites de las 4 columnas del dataset de ejemplo. Aquí va lo nuevo: el **Z-score**, el criterio para elegir entre los dos métodos, y las trampas que casi nunca se advierten.

### 41.1 Z-score — cuántas desviaciones estándar te alejas de la media

```text
z = (x − media) / desviación estándar
```

Convierte cualquier columna a una **escala común**: el valor deja de estar en pesos o en años y pasa a estar en "desviaciones estándar de distancia respecto al promedio".

```python
mean = df['price'].mean()
std  = df['price'].std()

df['z'] = (df['price'] - mean) / std

# Filtrar los candidatos
df[df['z'].abs() > 3]
```

💡 **Regla típica: `|z| > 3` → posible outlier.** Se usa el **valor absoluto** porque un dato puede estar muy lejos por arriba (z = +4) o por abajo (z = −4); ambos son atípicos.

### 🔁 Sintaxis genérica (copiar y adaptar)

```python
# ===== MÉTODO 1: IQR — para columnas SESGADAS =====
Q1  = df["nombre_columna"].quantile(0.25)
Q3  = df["nombre_columna"].quantile(0.75)
IQR = Q3 - Q1

lower = Q1 - 1.5 * IQR
upper = Q3 + 1.5 * IQR

outliers_iqr = df[(df["nombre_columna"] < lower) | (df["nombre_columna"] > upper)]
print("IQR      →", len(outliers_iqr), "outliers")

# ===== MÉTODO 2: Z-score — para columnas ~SIMÉTRICAS =====
mean = df["nombre_columna"].mean()
std  = df["nombre_columna"].std()

z = (df["nombre_columna"] - mean) / std   # Serie temporal, NO columna nueva

outliers_z = df[z.abs() > 3]
print("Z-score  →", len(outliers_z), "outliers")

# ===== Verificación =====
z.describe()   # mean ≈ 0 y std = 1 → la estandarización está bien hecha
```

**El procedimiento, paso a paso:**

| Paso | Qué haces | Código |
|---|---|---|
| 1. Diagnosticar la forma | ¿Sesgada o simétrica? Decide el método **antes** de calcular | `describe()`, media vs mediana (36.3), histograma (38) |
| 2. Calcular los límites | Vallas del IQR **o** media y `std` para el z | `.quantile()` · `.mean()` y `.std()` |
| 3. Filtrar los candidatos | Las filas fuera de los límites | `df[(df[col] < lower) \\| (df[col] > upper)]` · `df[z.abs() > 3]` |
| 4. Contar y contextualizar | **Cuántos** y **qué %** — un número suelto no dice nada | `len(outliers) / len(df) * 100` |
| 5. Decidir e investigar | ¿Error, sentinel o dato real? **No los borres** | Tabla de decisión de 39.6 |

⚠️ **`df["z"] = ...` deja la columna pegada al DataFrame para siempre.** Aparecerá en tu próximo `.info()`, en `.describe()` y en el CSV que exportes (sección 29). Si solo la necesitas para filtrar, usa una **Serie temporal** como en la plantilla de arriba (`z = ...` sin `df[]`). Si la creas como columna, bórrala después con `df = df.drop(columns="z")`.

#### Empaquetado en función (patrón de la sección 34.3)

```python
def detectar_outliers(df, col, metodo="iqr"):
    """Devuelve las filas atípicas de una columna según el método elegido."""
    if metodo == "iqr":
        q1  = df[col].quantile(0.25)
        q3  = df[col].quantile(0.75)
        iqr = q3 - q1
        lower = q1 - 1.5 * iqr
        upper = q3 + 1.5 * iqr
        return df[(df[col] < lower) | (df[col] > upper)]

    if metodo == "z":
        z = (df[col] - df[col].mean()) / df[col].std()
        return df[z.abs() > 3]

    return None
```

**Uso con un loop sobre varias columnas** (función + `for`, la sección 33.5):

```python
columnas_numericas = ["order_value", "customer_age", "price", "quantity"]

for col in columnas_numericas:
    atipicos = detectar_outliers(df, col, metodo="iqr")
    pct = len(atipicos) / len(df) * 100
    print(f"{col}: {len(atipicos)} outliers ({pct:.2f}%)")
```

💡 Cambia `metodo="iqr"` por `metodo="z"` y corre el mismo loop: comparar las dos salidas columna por columna es la forma más rápida de ver dónde discrepan los métodos (y por qué — sección 41.5).

⚠️ **Bug que circula mucho en código de ejemplo:** aparece

```python
df[df['z'].abs() > 3  # filtrar para ver registros
```

**Falta el corchete de cierre `]`.** Es `SyntaxError`. Debe ser `df[df['z'].abs() > 3]`.

💡 **Cómo verificar que estandarizaste bien.** Al hacer `df['z'].describe()` verás algo como:

```text
mean    -1.176836e-16
std      1.000000e+00
```

No te asustes con `-1.18e-16`: eso es **cero** en notación científica, con el error de redondeo normal de los decimales de punto flotante. Una columna estandarizada **siempre** tiene media 0 y desviación estándar 1 — si no te da eso, la fórmula está mal escrita.

### 41.2 De dónde sale el umbral 3 — la regla 68-95-99.7

| Dentro de… | Cae aproximadamente… | Queda fuera… |
|---|---|---|
| ±1 desviación | 68% | 32% |
| ±2 desviaciones | 95% | 5% |
| **±3 desviaciones** | **99.7%** | **0.3%** |

Por eso `|z| > 3` marca el 0.3% más extremo: en 5,000 filas serían \~15 registros.

⚠️ **Esos porcentajes valen SOLO si la distribución es normal.** En una distribución sesgada la regla 68-95-99.7 no se cumple, y el umbral 3 pierde su significado estadístico — sigue siendo un número, pero ya no equivale al "0.3% más extremo".

### 41.3 ⚠️ Por qué el Z-score sub-detecta outliers

Suele decirse que "el z-score sub-detecta outliers reales", pero no por qué. Hay **dos razones**, y las dos son importantes:

**1. Circularidad.** El z-score se calcula con la media y la `std`, y **ambas se contaminan con los propios outliers**. Un valor extremo infla la `std`, y como la `std` está en el denominador, **sube su propio umbral**. El outlier se esconde a sí mismo. El IQR no tiene este problema porque Q1 y Q3 no se mueven con los extremos (sección 39.1).

**2. Techo por tamaño de muestra.** Con `n` datos y la `std` muestral de pandas, el z-score máximo posible está acotado:

```text
|z| máximo posible ≤ (n − 1) / √n
```

| n | `\|z\|` máximo posible | ¿Puede detectar algo con el umbral 3? |
|---|---|---|
| 10 | 2.85 | **Nunca** — matemáticamente imposible |
| 50 | 6.93 | Sí, pero muy restringido |
| 5,000 | 70.7 | Sí, sin problema |

🎯 **Consecuencia práctica:** con muestras pequeñas, "no encontré outliers con z-score" puede significar simplemente que **no cabían**. Antes de concluir, revisa cuántas filas tienes.

### 41.4 ⚠️ La trampa del `.head(50)` en la demostración

La lección arranca con `df = pd.read_csv(...).head(50)` para "interpretar mejor los resultados". Ahí empiezan los problemas:

| Dato | En `.head(50)` | En el dataset completo (36.7) |
|---|---|---|
| Q1 de `price` | 267.25 | 218 |
| Q3 de `price` | 580 | 847.25 |
| IQR | 312.75 | 629.25 |
| `std` | — | **1,173.27** |
| `max` de `price` | — | 36,708 |

**Error 1 — mezclar los dos conjuntos.** Es frecuente ver que, tras declarar `.head(50)`, se muestra `df['price'].std()` con resultado **1173.2651824540399**. Ese es el valor del dataset **completo** (la tabla de 36.7), no el de 50 filas. Verifica siempre de qué DataFrame sale cada número.

**Error 2 — la conclusión "no hay outliers" es un artefacto del recorte.** Con las 50 primeras filas, el `|z|` máximo es 2.73 y no se detecta nada. Sobre la columna completa:

```text
z del máximo = (36,708 − 756.39) / 1,173.27 ≈ 30.6
```

**Un z de 30.** No es que el z-score falle en `price`: es que en 50 filas no estaba ningún valor extremo. Es un problema de muestreo presentado como si fuera una propiedad del método.

| Columna | z del máximo (dataset completo) |
|---|---|
| `price` | 30.6 |
| `order_value` | 23.7 |
| `quantity` | 22.0 |
| `customer_age` | 1.7 |

📘 **Regla que te llevas:** nunca calcules umbrales de outliers sobre un subconjunto y los apliques al total. Los cuartiles, la media y la `std` **cambian con la muestra**; los límites que salgan de ahí no son válidos fuera de ella.

### 41.5 IQR vs Z-score — cuál usar

|   | **IQR** (1.5 × IQR) | **Z-score** (`\|z\| > 3`) |
|---|---|---|
| Se basa en | Cuartiles (robustos) | Media y `std` (contaminables) |
| Ideal para | Datos **sesgados**, colas largas | Distribuciones **aproximadamente normales** |
| Falla cuando | Hay **múltiples modas** (bimodal) | Hay sesgo fuerte → **sub-detecta** |
| Riesgo | Marca **demasiados** valores como atípicos | Marca **muy pocos** |

**Aplicado al dataset de ejemplo:**

| Columna | Forma | Método recomendado |
|---|---|---|
| `price` | Sesgo fuerte a la derecha | **IQR** — el z-score se distorsiona con la propia cola |
| `quantity` | Sesgo fuerte a la derecha | **IQR** |
| `order_value` | **Bimodal**  • cola larga | ⚠️ Ninguno de los dos limpiamente — ver abajo |
| `customer_age` | Uniforme (sintética) | Irrelevante — no hay outliers posibles |

⚠️ **Contradicción frecuente en `order_value`.** Suele declararse como limitación del IQR que *"no funciona bien si la distribución tiene múltiples modas (bimodal)"* y **acto seguido** recomienda IQR para `order_value`. Pero `order_value` **es bimodal**: se deduce en 36.7 (media 10,075 por debajo de la mediana 10,341 pese a un máximo de 303,824) y se ve en el histograma de 40.2, que tiene dos picos. Con dos poblaciones mezcladas, la caja se estira para cubrir ambas y el IQR se infla: los límites salen demasiado anchos y dejan pasar atípicos reales de la población baja.

💡 **Qué hacer ahí:** separa las dos poblaciones primero (filtra por el punto donde cae el valle entre los picos) y calcula los límites **dentro de cada grupo**. Es el mismo razonamiento de comparar métricas por grupo de la sección 22.

### 41.6 `customer_age`: que los dos métodos coincidan aquí no prueba nada

Un cierre habitual aplica ambos métodos a `customer_age`, ninguno detecta outliers, y concluye: *"vimos dos métodos distintos que llegan al mismo resultado"*, como si fuera una validación cruzada.

**No lo es.** Esa columna es **uniforme entre 18 y 80** (36.7) y por tanto **no tiene cola**. Ningún método puede encontrar ahí un valor atípico, porque no existen:

- **IQR:** valla superior 109 contra un máximo de 80 (tabla de 39.3).
- **Z-score:** `|z|` máximo = (80 − 49.12) / 17.71 = **1.74**, muy por debajo de 3.

Dos métodos coincidiendo sobre una columna sin extremos no demuestra que los métodos funcionen — demuestra que la columna no tiene nada que detectar. Para comparar de verdad los dos métodos hay que correrlos sobre `price`, donde **sí** discrepan.

📌 **Es la tercera vez que esta columna sintética invita a una conclusión falsa.** Ver 36.7 (uniformidad), 39.3 (cero outliers posibles) y 40.3 (el pico de imputación).

### Términos clave

**Z-score / puntaje Z** (`(x − media) / std`) · Estandarización (media 0, `std` 1) · `.abs()` · Umbral `|z| > 3` · **Regla 68-95-99.7** (empírica, solo para distribución normal) · Circularidad del z-score (el outlier infla su propio umbral) · Techo `(n−1)/√n` · Método **robusto** (IQR) vs **no robusto** (z-score) · Sub-detección vs sobre-marcado · Notación científica (`-1.18e-16` = cero).

📌 **Conecta con:** sección 22 (comparar por grupo), 36.7 (cuartiles, media y `std` de cada columna), 39.1 a 39.3 (IQR, vallas y la tabla de límites), 39.6 (qué hacer con un outlier: no borrarlo), 40.2 y 40.3 (bimodalidad de `order_value`, artefacto de `customer_age`).

---

## 42. Qué hacer con un outlier — KEEP, DROP y CAP (winsorización)

**Cuándo usarlo:** después de detectarlos (39 y 41). Detectar es mecánico; **decidir es el trabajo analítico**.

📌 **La tabla de decisión básica ya está en la sección 39.6.** Aquí va lo nuevo: las tres estrategias con nombre propio, la **winsorización** completa con `np.clip()`, cómo elegir el umbral y —lo más importante— **cuándo no debes capar**.

### 42.1 Las tres estrategias

| Estrategia | Cuándo | Ejemplo | Riesgo de equivocarte |
|---|---|---|---|
| **KEEP** (conservar) | El valor es **real** y representa comportamiento legítimo | Clientes *whales* con `order_value` alto; productos premium; picos de Black Friday | Si lo borras, destruyes la segmentación VIP y el revenue real |
| **DROP** (eliminar) | Certeza **absoluta** de que es un error imposible | `quantity <= 0`; sentinel `-999` en `customer_age`; fechas fuera del periodo válido | Si lo conservas, envenena media y `std` |
| **CAP / Winsorizar** | El valor es **posible pero muy raro**, y distorsiona las métricas | `order_value` extremos que inflan el promedio | Si capas de más, borras justo el fenómeno que ibas a estudiar |

🎯 **La regla mental:** *KEEP* para los VIP y eventos reales · *DROP* para los imposibles · *CAP* para moderar lo muy raro sin perderlo.

⚠️ **Ojo con los ejemplos de DROP que ya no aplican.** Se suele mencionar `customer_age = -999` y `quantity <= 0`, pero a esta altura del flujo **ya están limpios**: en el `describe()` de esta misma sección, `customer_age` tiene `min` = 18 y `quantity` tiene `min` = 1. Ese DROP pertenece a las secciones 31.4 y 35.2, no a esta etapa.

### 42.2 Winsorización con `np.clip()`

**Winsorizar = sustituir el valor extremo por el límite**, en vez de borrar la fila. El dato no se pierde: se mueve al borde.

```python
import numpy as np
import pandas as pd

df = pd.read_csv("data/ventas_limpio.csv")

# 1) Definir los límites por percentil
lower = df['order_value'].quantile(0.01)   # percentil 1
upper = df['order_value'].quantile(0.99)   # percentil 99

# 2) Capar en una columna NUEVA (nunca sobre la original)
df['order_value_winsor'] = np.clip(df['order_value'], lower, upper)

# 3) Comparar
df[["order_value", "order_value_winsor"]].describe()
```

**Qué hace `np.clip(serie, lower, upper)`:** fuerza cada valor a quedarse dentro del rango. Si es menor que `lower` → lo vuelve `lower`. Si es mayor que `upper` → lo vuelve `upper`. Lo demás queda intacto.

💡 **No necesitas NumPy para esto.** Pandas tiene el mismo método directamente en la Serie:

```python
df['order_value_winsor'] = df['order_value'].clip(lower, upper)
```

Hacen lo mismo. La versión de pandas evita un `import` y encadena mejor con el resto de tu flujo.

🎯 **Columna nueva, siempre.** Crea `order_value_winsor` en vez de pisar `order_value`: conservas el original para el revenue real y usas el capado para las métricas que necesitan estabilidad. Es el mismo principio de la **sección 5** (copia de seguridad antes de limpiar).

### 🔁 Sintaxis genérica (copiar y adaptar)

```python
# Winsorizar por percentiles
lower = df["nombre_columna"].quantile(0.01)
upper = df["nombre_columna"].quantile(0.99)

df["nombre_columna_winsor"] = df["nombre_columna"].clip(lower, upper)

# ¿Cuántos valores se movieron realmente?
movidos = (df["nombre_columna"] != df["nombre_columna_winsor"]).sum()
print(movidos, "valores capados —", round(movidos / len(df) * 100, 2), "%")
```

### 42.3 Elegir el umbral: percentiles vs vallas del IQR

Es común ver P1/P99 sin explicación, justo después de haber presentado las vallas de 1.5 × IQR. Son **dos criterios distintos** para el mismo trabajo:

|   | Percentiles (P1 / P99) | Vallas IQR (`Q3 + 1.5·IQR`) |
|---|---|---|
| Cuántos valores capa | **Exactamente el 1% por lado, por construcción** | Los que haya fuera — pueden ser 0 o el 15% |
| Depende de | Una decisión tuya sobre cuánto recortar | La forma real de la distribución |
| Útil cuando | Quieres un recorte controlado y predecible | Quieres respetar el criterio estadístico de "atípico" |

⚠️ **Ojo con el P99: capa el 1% superior *siempre*, existan outliers o no.** Si aplicas P1/P99 a `customer_age` —que es uniforme y no tiene un solo atípico (39.3)— igual te va a modificar 100 registros. El percentil no pregunta si el valor es raro; solo cuenta posiciones.

```python
# Compara los dos umbrales antes de decidir
Q1  = df["order_value"].quantile(0.25)
Q3  = df["order_value"].quantile(0.75)
valla_iqr = Q3 + 1.5 * (Q3 - Q1)
p99       = df["order_value"].quantile(0.99)

print("Valla IQR:", valla_iqr, "| P99:", p99)
print("Fuera de la valla:", (df["order_value"] > valla_iqr).sum())
print("Por encima de P99:", (df["order_value"] > p99).sum())
```

📌 De tu tabla de **39.3**, la valla del IQR de `order_value` está en **28,260**. Compárala con el P99 que salga: si el P99 es mucho mayor, capar en P99 deja pasar atípicos que el IQR sí marcaba.

### 42.4 Evaluación pre–post: obligatoria

📌 **Retoma la sección 31.5.** Capar cambia las métricas — ese es el objetivo. Pero tienes que **cuantificar cuánto**, o no sabrás si cambiaste la conclusión:

```python
comparacion = pd.DataFrame({
    "antes":   df["order_value"].describe(),
    "despues": df["order_value_winsor"].describe()
})
comparacion["cambio_pct"] = ((comparacion["despues"] - comparacion["antes"])
                             / comparacion["antes"] * 100).round(2)
comparacion
```

| Qué mirar | Qué esperar |
|---|---|
| `mean` | **Baja** — es el efecto buscado |
| `std` | **Baja bastante** — la varianza era casi toda de la cola |
| `median` | **No se mueve** — si se movió, capaste demasiado |
| `max` | Ahora es exactamente `upper` |
| `count` | **Idéntico** — winsorizar no borra filas |

🎯 **La mediana es tu control de seguridad.** Si al winsorizar la mediana se movió, tocaste el 50% central: eso ya no es capar la cola, es deformar la distribución.

### 42.5 ⚠️ Cuándo NO winsorizar

La winsorización suele venderse como un punto medio cómodo. Tiene costos reales:

| Si tu objetivo es… | Winsorizar… |
|---|---|
| Calcular **ingresos totales** | ❌ Destruye la cifra. Capar reduce el `sum()`: estarías reportando menos dinero del que entró |
| **Identificar clientes VIP / whales** | ❌ Borra justo lo que buscas. Todos los grandes quedan igualados en el tope |
| **Detectar fraude o anomalías** | ❌ El caso sospechoso se disfraza de normal |
| Describir el **comportamiento típico** | ✅ Tiene sentido — o usa directamente la mediana (36.3) |
| Alimentar un **modelo sensible a la varianza** | ✅ Es su uso clásico |

⚠️ **El problema de las columnas relacionadas.** Si winsorizas `order_value` pero no `price` ni `quantity`, y esas columnas están ligadas entre sí, el DataFrame queda **internamente inconsistente**: habrá filas donde el valor capado ya no cuadra con su precio y su cantidad. Antes de decidir, comprueba la relación:

```python
# ¿order_value es realmente price * quantity?
(df["price"] * df["quantity"] == df["order_value"]).mean()
```

Si esa proporción es alta, **cualquier capado tiene que ser coherente entre las tres columnas** o documentado como una columna auxiliar que no se usa para recomponer totales.

### 42.6 ⚠️ Dos cosas en la salida que casi nadie comenta

**1. `df.median()` sobre todo el DataFrame mezcla IDs y flags con datos reales.**

```text
order_id                  2500.5      ← sin sentido
customer_id               1988.0      ← sin sentido
quantity_invalid_flag        1.0      ← ver abajo
```

La mediana de `order_id` es 2500.5 porque son 5,000 IDs consecutivos. **No es información, es aritmética.** Es exactamente la distinción de las secciones 30.1 y 37.3: un ID es categórico aunque sea numérico. Calcula las medidas sobre una **lista explícita** de columnas, como haces en 36.7, no sobre el DataFrame completo.

**2. `quantity_invalid_flag` tiene mediana 1.0. Eso merece que lo revises.**

Una columna binaria (0/1) solo puede tener mediana 1.0 si **al menos la mitad de las filas valen 1**. Si esa bandera marca los registros con cantidad inválida —que es lo que hace `crear_flags` en la sección **35.3**— estaría diciendo que **más del 50% del dataset tenía `quantity` inválida**, y eso no cuadra con lo que documentaste al limpiar.

Hay tres explicaciones posibles y conviene distinguirlas:

```python
df["quantity_invalid_flag"].value_counts()   # ¿cuántos 1 y cuántos 0?
df["age_invalid_flag"].value_counts()        # las otras dos dan mediana 0.0
df["state_missing_flag"].value_counts()
```

1. La bandera está **invertida** (1 = válido) → renombrar, porque el nombre miente.
2. La bandera marca algo **más amplio** de lo que su nombre sugiere.
3. `crear_flags` tiene un bug y marca casi todo → revisar la condición en 35.3.

📌 Sea cual sea la respuesta, es un dato que va a tu narrativa de portafolio: las banderas de missingness son parte de la evidencia de limpieza, y una que marca al 50% del dataset no se sostiene sin explicación.

### Términos clave

**KEEP / DROP / CAP** · **Winsorización** (capar, no eliminar) · `np.clip(serie, lower, upper)` · `.clip()` (método de pandas, sin NumPy) · Umbral por **percentil** (P1/P99) vs **valla IQR** · Columna paralela (`_winsor`) vs sobrescribir · Evaluación pre–post (31.5) · **La mediana como control de seguridad** · Consistencia entre columnas relacionadas · Bandera de missingness con mediana 1.0.

📌 **Conecta con:** sección 5 (copia antes de limpiar), 14 (valores imposibles), 29 (exportar), 30.1 (rol vs tipo), 31.4 y 31.5 (limpiar y evaluar), 35.3 (`crear_flags`), 36.3 (mediana como valor típico), 37.3 (ID numérico ≠ variable numérica), 39.3 y 39.6 (vallas y decisión), 41 (detección con IQR y z-score).

---

[← Distribuciones e histogramas](08-distribuciones.md) · [Índice](../README.md) · [Feature engineering →](10-feature-engineering.md)
