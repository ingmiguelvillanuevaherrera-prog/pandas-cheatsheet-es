# Sintaxis genérica — todo junto

Todos los bloques **🔁 Sintaxis genérica** de la guía, extraídos y ordenados por sección.
Cada uno usa `nombre_columna` como marcador: sustitúyelo por el tuyo y funciona.

Para el contexto (cuándo usar cada uno, qué validar después, qué puede salir mal), abre la sección enlazada.

---

### 4. Renombrar, eliminar, reordenar

[→ contexto completo](../docs/01-carga-e-inspeccion.md#4-renombrar-eliminar-reordenar)

```python
# Renombra una o más columnas
df = df.rename(columns={"nombre_viejo": "nombre_nuevo"})

# Elimina columnas
df.drop(["columna_1", "columna_2"], axis=1, inplace=True)

# Conserva solo ciertas columnas (y las reordena)
df = df[["columna_1", "columna_2", "columna_3"]]
```

### 6. Diagnóstico de nulos

[→ contexto completo](../docs/02-nulos-y-duplicados.md#6-diagnóstico-de-nulos)

```python
# Cuenta cada valor de 'nombre_columna', incluyendo los NaN
df["nombre_columna"].value_counts(dropna=False)
```

### 8. Opción A — Eliminar nulos: `.dropna()`

[→ contexto completo](../docs/02-nulos-y-duplicados.md#8-opción-a-eliminar-nulos-dropna)

```python
# Elimina filas donde 'nombre_columna' es nula y renumera el índice
df = df.dropna(subset=["nombre_columna"]).reset_index(drop=True)
```

### 13. Eliminar duplicados

[→ contexto completo](../docs/02-nulos-y-duplicados.md#13-eliminar-duplicados)

```python
# Cuenta filas duplicadas exactas (fila completa)
df.duplicated().sum()

# Cuenta llaves repetidas según columnas clave
keys = ["columna_1", "columna_2"]
df.duplicated(subset=keys).sum()

# Elimina duplicados por llave, conserva la primera aparición y renumera
df = df.drop_duplicates(subset=keys, keep="first").reset_index(drop=True)
```

### 14. Filtrar filas irrelevantes (valores imposibles)

[→ contexto completo](../docs/02-nulos-y-duplicados.md#14-filtrar-filas-irrelevantes-valores-imposibles)

```python
# Cuenta filas que violan la regla de plausibilidad
(df["nombre_columna"] <= valor_limite).sum()

# Conserva solo filas válidas y renumera el índice
df = df[df["nombre_columna"] > valor_limite].reset_index(drop=True)
```

### 15. Auditar y validar la limpieza

[→ contexto completo](../docs/02-nulos-y-duplicados.md#15-auditar-y-validar-la-limpieza)

```python
df["nombre_columna"].nunique()                    # cuántos valores únicos hay
df["nombre_columna"].value_counts()               # conteo de cada valor
df.duplicated(subset=["col_1", "col_2"]).sum()    # llaves repetidas (debe dar 0)
```

### 16. Estandarizar texto (normalizar strings)

[→ contexto completo](../docs/03-tipos-texto-y-fechas.md#16-estandarizar-texto-normalizar-strings)

```python
# Limpia texto: quita espacios y unifica mayúsculas, guardando en la misma columna
df["nombre_columna"] = df["nombre_columna"].str.strip().str.upper()

# Filtra filas donde 'nombre_columna' contiene un patrón
filtro = df[df["nombre_columna"].str.contains("ejemplo")]
```

### 18. Convertir texto a números (flujo completo)

[→ contexto completo](../docs/03-tipos-texto-y-fechas.md#18-convertir-texto-a-números-flujo-completo)

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

### 19. Convertir fechas a datetime

[→ contexto completo](../docs/03-tipos-texto-y-fechas.md#19-convertir-fechas-a-datetime)

```python
# Convierte 'nombre_columna' a fecha de forma segura (inválidos → NaT)
df["nombre_columna"] = pd.to_datetime(df["nombre_columna"], errors="coerce", utc=True)

# Extrae componentes en columnas nuevas
df["anio"] = df["nombre_columna"].dt.year
df["mes"] = df["nombre_columna"].dt.month
```

### 21. Filtrado avanzado de filas

[→ contexto completo](../docs/04-filtrado-y-estadistica.md#21-filtrado-avanzado-de-filas)

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

### 22. Estadísticas resumidas (descriptivas)

[→ contexto completo](../docs/04-filtrado-y-estadistica.md#22-estadísticas-resumidas-descriptivas)

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

### 23. Agrupar y agregar — `.groupby()`

[→ contexto completo](../docs/04-filtrado-y-estadistica.md#23-agrupar-y-agregar-groupby)

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

### 24. Ordenar datos — `.sort_values()`

[→ contexto completo](../docs/04-filtrado-y-estadistica.md#24-ordenar-datos-sort_values)

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

### 25. Preparar datasets para unirlos (pre-merge)

[→ contexto completo](../docs/05-merge-y-exportacion.md#25-preparar-datasets-para-unirlos-pre-merge)

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

### 26. Validación visual (matplotlib + seaborn)

[→ contexto completo](../docs/05-merge-y-exportacion.md#26-validación-visual-matplotlib-seaborn)

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

### 26. Validación visual (matplotlib + seaborn)

[→ contexto completo](../docs/05-merge-y-exportacion.md#26-validación-visual-matplotlib-seaborn)

```python
# Dos variables de escalas distintas en un gráfico de barras
df.plot(x="columna_categoria", y=["columna_chica", "columna_grande"], kind="bar",
        secondary_y="columna_grande", figsize=(12, 5))
plt.title("Título")
plt.xticks(rotation=90)   # rota las etiquetas del eje X si son largas
plt.show()
```

### 27. Combinar datasets — `pd.merge()`

[→ contexto completo](../docs/05-merge-y-exportacion.md#27-combinar-datasets-pdmerge)

```python
# Estructura: merge(tabla_izquierda, tabla_derecha, on=claves, how=tipo)
df_unido = pd.merge(df_izq, df_der, on=["clave_1", "clave_2"], how="inner")

# Left join: conserva TODO el df de la izquierda
df_unido = pd.merge(df_principal, df_extra, on=["clave_1", "clave_2"], how="left")
```

### 29. Exportar resultados — `.to_csv()`

[→ contexto completo](../docs/05-merge-y-exportacion.md#29-exportar-resultados-to_csv)

```python
# Exportar a CSV sin la columna de índice
df.to_csv("nombre_archivo_limpio.csv", index=False)
```

### 30. Revisión inicial: roles, faltantes e inválidos (diagnóstico de calidad)

[→ contexto completo](../docs/06-calidad-de-datos.md#30-revisión-inicial-roles-faltantes-e-inválidos-diagnóstico-de-calidad)

```python
# Valores únicos por columna, ordenados de menor a mayor
df.nunique().sort_values()
```

### 30. Revisión inicial: roles, faltantes e inválidos (diagnóstico de calidad)

[→ contexto completo](../docs/06-calidad-de-datos.md#30-revisión-inicial-roles-faltantes-e-inválidos-diagnóstico-de-calidad)

```python
# Proporción de nulos por columna (0 a 1), mayor a menor
df.isna().mean().sort_values(ascending=False)

# En porcentaje, redondeado
(df.isna().mean() * 100).round(2).sort_values(ascending=False)
```

### 30. Revisión inicial: roles, faltantes e inválidos (diagnóstico de calidad)

[→ contexto completo](../docs/06-calidad-de-datos.md#30-revisión-inicial-roles-faltantes-e-inválidos-diagnóstico-de-calidad)

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

### 31. Manejar valores ausentes e inválidos (MCAR/MAR/MNAR + imputación)

[→ contexto completo](../docs/06-calidad-de-datos.md#31-manejar-valores-ausentes-e-inválidos-mcarmarmnar-imputación)

```python
# Proporción de faltantes de una columna, desglosada por grupo
df["columna_con_nulos"].isna().groupby(df["columna_grupo"]).mean().sort_values(ascending=False)
```

### 31. Manejar valores ausentes e inválidos (MCAR/MAR/MNAR + imputación)

[→ contexto completo](../docs/06-calidad-de-datos.md#31-manejar-valores-ausentes-e-inválidos-mcarmarmnar-imputación)

```python
df["flag_vacio"] = df["nombre_columna"].isna().astype(int)
df.groupby("flag_vacio")["columna_comportamiento"].describe()
```

### 31. Manejar valores ausentes e inválidos (MCAR/MAR/MNAR + imputación)

[→ contexto completo](../docs/06-calidad-de-datos.md#31-manejar-valores-ausentes-e-inválidos-mcarmarmnar-imputación)

```python
# Marcar inválidos como NaN
df.loc[df["nombre_columna"] <= valor_limite, "nombre_columna"] = np.nan

# Validar una fórmula de reconstrucción
df["col_calculada"] = df["col_a"] / df["col_b"]
df[["col_a", "col_b", "nombre_columna", "col_calculada"]].head()

# Imputar SOLO los NaN con la fórmula validada
df["nombre_columna"] = df["nombre_columna"].fillna(df["col_a"] / df["col_b"])
```

### 31. Manejar valores ausentes e inválidos (MCAR/MAR/MNAR + imputación)

[→ contexto completo](../docs/06-calidad-de-datos.md#31-manejar-valores-ausentes-e-inválidos-mcarmarmnar-imputación)

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

### 32. Funciones para automatizar la limpieza (`def`)

[→ contexto completo](../docs/07-python-para-limpieza.md#32-funciones-para-automatizar-la-limpieza-def)

```python
# 1) DEFINIR (esto solo la guarda)
def nombre_de_la_funcion(parametro):
    # lo que hace, siempre indentado
    print(parametro)

# 2) LLAMAR (esto sí la ejecuta)
nombre_de_la_funcion(valor)
```

### 33. Bucles (`for` & `while`) e iterables en una colección

[→ contexto completo](../docs/07-python-para-limpieza.md#33-bucles-for-while-e-iterables-en-una-colección)

```python
columnas_a_revisar = ["col_a", "col_b", "col_c"]

for col in columnas_a_revisar:
    print(col, "nulos:", df[col].isna().sum())
    # sustituye la línea de arriba por cualquier regla:
    # conteo de "?", validación de tipo, etc.
```

### 34. Mini-funciones de limpieza — un loop DENTRO de una función

[→ contexto completo](../docs/07-python-para-limpieza.md#34-mini-funciones-de-limpieza-un-loop-dentro-de-una-función)

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

### 36. Diagnosticar la forma de la distribución (media y mediana sobre el histograma)

[→ contexto completo](../docs/08-distribuciones.md#36-diagnosticar-la-forma-de-la-distribución-media-y-mediana-sobre-el-histograma)

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

### 45. Comparar una distribución entre grupos (`hue` en histplot)

[→ contexto completo](../docs/08-distribuciones.md#45-comparar-una-distribución-entre-grupos-hue-en-histplot)

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

### 41. Detectar outliers con reglas estadísticas — IQR y Z-score

[→ contexto completo](../docs/09-outliers.md#41-detectar-outliers-con-reglas-estadísticas-iqr-y-z-score)

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

### 42. Qué hacer con un outlier — KEEP, DROP y CAP (winsorización)

[→ contexto completo](../docs/09-outliers.md#42-qué-hacer-con-un-outlier-keep-drop-y-cap-winsorización)

```python
# Winsorizar por percentiles
lower = df["nombre_columna"].quantile(0.01)
upper = df["nombre_columna"].quantile(0.99)

df["nombre_columna_winsor"] = df["nombre_columna"].clip(lower, upper)

# ¿Cuántos valores se movieron realmente?
movidos = (df["nombre_columna"] != df["nombre_columna_winsor"]).sum()
print(movidos, "valores capados —", round(movidos / len(df) * 100, 2), "%")
```
