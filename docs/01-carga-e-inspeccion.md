# Carga e inspección inicial

[← Índice de la guía](../README.md)

---

## 1. Cargar datos

```python
import pandas as pd                      # pd = alias estándar

df = pd.read_csv("ruta/archivo.csv")     # CSV
df = pd.read_excel("archivo.xlsx")       # Excel
```

⚠️ **Carga tus datos UNA sola vez.** Si vuelves a ejecutar `read_csv` más abajo, pierdes toda la limpieza.

---

## 2. Inspección rápida

| Comando | Qué hace | ¿Paréntesis? |
|---|---|---|
| `df.head()` | Primeras 5 filas (o `head(n)`) | Sí — método |
| `df.tail()` | Últimas 5 filas | Sí |
| `df.sample(3)` | Filas aleatorias | Sí |
| `df.shape` | Tupla `(filas, columnas)` | **No — atributo** |
| `df.info()` | Tipos, no-nulos, memoria | Sí |
| `df.describe()` | Estadísticas de columnas numéricas | Sí |
| `df.columns` | Nombres de columnas | **No — atributo** |
| `df.dtypes` | Tipo de cada columna | **No — atributo** |
| `df[col].dtype` | Tipo de UNA columna | No |

**Truco para no confundir:** método = acción = lleva `()`. Atributo = propiedad = sin `()`.

### `.describe()` según tipo de columna

- **Numérica** → count, mean, std, min, 25%, 50% (mediana), 75%, max
- **Texto** → count, unique (valores distintos), top (más frecuente), freq (veces que aparece el top) cou

### Percentiles (recordatorio)

El percentil 75 = valor bajo el cual está el 75% de los datos. Si el 75% está en 20 pero el máximo es 90 → **posibles outliers**.

---

## 3. Seleccionar columnas

```python
df["columna"]                            # UNA columna → 1 par de corchetes
df[["col1", "col2", "col3"]]             # VARIAS columnas → 2 pares (lista adentro)
```

El doble corchete también **reordena**: las columnas quedan en el orden de la lista.

---

## 4. Renombrar, eliminar, reordenar

```python
# Renombrar (diccionario: {"viejo": "nuevo"})
df = df.rename(columns={"value": "pm25_level"})

# Eliminar columnas explícitamente
df.drop(["X", "Y"], axis=1, inplace=True)
#        lista       axis=1 = columnas   inplace=True = guarda sin reasignar

# Quedarte SOLO con algunas (selección + reorden)
df = df[["country_id", "pm25_level", "unit"]]
```

⚠️ **Guarda el cambio:** si no usas `inplace=True`, necesitas `df = df.rename(...)`. Sin la asignación, el cambio se pierde.

**🔁 Sintaxis genérica:**

```python
# Renombra una o más columnas
df = df.rename(columns={"nombre_viejo": "nombre_nuevo"})

# Elimina columnas
df.drop(["columna_1", "columna_2"], axis=1, inplace=True)

# Conserva solo ciertas columnas (y las reordena)
df = df[["columna_1", "columna_2", "columna_3"]]
```

|   | `df.drop()` | `df[[...]]` |
|---|---|---|
| Uso | Quitar columnas no deseadas | Mantener solo las listadas |

---

## 5. Antes de limpiar: copia de seguridad

```python
df_clean = df.copy()      # ← CON paréntesis, es un método
```

- `df` → original intacto
- `df_clean` → donde trabajas

---

[Índice](../README.md) · [Valores nulos y duplicados →](02-nulos-y-duplicados.md)
