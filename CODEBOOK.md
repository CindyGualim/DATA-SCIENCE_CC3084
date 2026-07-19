
**Proyecto 1 — Obtención y Limpieza de los Datos**
CC3084 Data Science · Universidad del Valle de Guatemala · Semestre II 2026 · Grupo 2

| Integrante | Carné | Incisos de la actividad 5 |
|---|---|---|
| Jose Donado | 22684 | *(completar según reparto)* |
| Daniela Ramírez | 23053 | *(completar según reparto)* |
| Cindy Gualim | 21226 | *(completar según reparto)* |

---

## 1. Metadatos generales

| Campo | Valor |
|---|---|
| **Fuente de los datos** | Sistema de Búsqueda de Establecimientos Educativos, MINEDUC Guatemala — http://www.mineduc.gob.gt/BUSCAESTABLECIMIENTO_GE/ |
| **Método de obtención** | Web scraping con Selenium sobre el formulario público (`cmbDepartamento`, `cmbMunicipio`, `cmbNivel`, `cmbSector`, `ddlplan`, `ddlModalidad`), consultando los 22 departamentos con Municipio / Sector / Plan / Modalidad = `TODOS` |
| **Fecha de extracción del crudo** | `AAAA-MM-DD` — **completar con la fecha real en que se corrió el scraping** |
| **Fecha de esta versión del libro de códigos** | `AAAA-MM-DD` |
| **Nivel escolar** | DIVERSIFICADO (único nivel exigido por la consigna) |
| **Cobertura geográfica** | Los 22 departamentos de Guatemala |
| **Unidad de observación** | Un establecimiento educativo con oferta de nivel Diversificado |
| **Notebook de origen** | `Proyecto1.ipynb` (actividades 1 a 10 de la guía) |
| **Archivo crudo** | `establecimientos.csv` |
| **Archivo crudo filtrado** | `data/raw/establecimientos_diversificado_raw.csv` |
| **Archivo limpio final** | `data/clean/establecimientos_diversificado_limpio.csv` |
| **Versión del conjunto limpio** | v1.0 |
| **Codificación** | UTF-8 con BOM (`utf-8-sig`) |
| **Registros (crudo filtrado → limpio)** | `____` → `____` *(completar con la corrida final)* |
| **Variables** | 26 (17 originales + 9 derivadas) |

### Estructura del repositorio

```
DATA-SCIENCE_CC3084/
├── data/
│   ├── raw/    establecimientos_diversificado_raw.csv
│   └── clean/  establecimientos_diversificado_limpio.csv
├── establecimientos.csv          <- crudo tal como sale del scraping
├── CODEBOOK.md                   <- este documento
└── Proyecto1.ipynb               <- proceso completo y reproducible
```

Dependencias: `pandas`, `numpy`, `rapidfuzz`, `selenium`, `webdriver-manager`
(las dos últimas solo para las actividades 1 y 2).

---

## 2. Plan de limpieza (actividad 4)

Este plan se elaboró **antes** de modificar los datos, a partir del diagnóstico de la actividad 3.
Para cada variable documenta tres cosas: el problema detectado, la regla que se usaría para
corregirlo (y por qué debería funcionar) y los riesgos de aplicar esa regla.

**Cómo leerlo.** La columna de riesgos es la más importante: deja constancia de qué información
podría dañarse o perderse con cada transformación, para que quien use el conjunto limpio sepa qué
supuestos se hicieron. La ejecución real de este plan está en la actividad 5 del notebook, y lo que
efectivamente cambió (con el número de registros afectados) queda en el registro de
transformaciones de la actividad 6.

| Variable | Problema encontrado | Regla de corrección | Riesgos |
|---|---|---|---|
| `CODIGO` | Faltantes provenientes de filas 100 % vacías. Formato ya consistente. | Eliminar las filas vacías; conservar el código como texto, no como número. | Convertirlo a numérico perdería los guiones y los ceros a la izquierda. |
| `DISTRITO` | Faltantes; dos formatos coexistentes (`NN-NNN` y `NN-NN-NNNN`); valores truncados. | Registrar el formato en una variable derivada y pasar los truncados a NA. | No hay regla segura para convertir un formato al otro sin inventar dígitos. |
| `DEPARTAMENTO` | Solo faltantes de filas vacías; dominio limpio. | Validar contra el catálogo oficial de 22 departamentos. | Ninguno relevante. |
| `MUNICIPIO` | No cubre los 340 municipios oficiales; posibles variantes de escritura. | Normalizar a mayúsculas y buscar variantes por similitud de cadenas **dentro de cada departamento**. | Hay municipios homónimos en departamentos distintos: unificarlos entre departamentos sería un error grave. |
| `ESTABLECIMIENTO` | Comillas simples y dobles mezcladas; comillas sin cerrar. | Unificar a comillas dobles y eliminar comillas sueltas al inicio o al final. | Cambio cosmético; se documenta que no altera el nombre real del establecimiento. |
| `DIRECCION` | Minúsculas mezcladas, espacios múltiples, abreviaturas de ordinal inconsistentes. | Normalizar espacios y pasar a mayúsculas. | El texto libre puede contener nombres propios cuya capitalización original se pierde. |
| `TELEFONO` | Alta tasa de faltantes; varios números por celda; longitudes fuera de 8 dígitos; texto mezclado. | Extraer los números de 8 dígitos: el primero a `telefono`, el segundo a `telefono_2`; NA si ninguno califica. | **Riesgo más alto del proyecto:** se puede descartar el número realmente vigente y conservar uno obsoleto. |
| `SUPERVISOR` | Faltantes; posibles variantes de escritura del mismo nombre. | Normalizar texto; señalar variantes por similitud **sin fusionarlas** automáticamente. | Personas distintas con nombres parecidos podrían fusionarse por error. |
| `DIRECTOR` | La variable con más faltantes; mismo riesgo que `SUPERVISOR`. | Igual que `SUPERVISOR`; documentar que el faltante puede ser una vacante real. | Asumir que "falta" equivale a "error de captura" sería incorrecto. |
| `NIVEL` | Constante tras el filtro a DIVERSIFICADO. | Conservar como categórica; sirve de control del filtro aplicado. | Ninguno. |
| `SECTOR` | Dominio limpio. | Normalizar texto y validar las categorías observadas. | Ninguno. |
| `AREA` | Categoría `SIN ESPECIFICAR`, fuera del dominio binario {URBANA, RURAL}. | Recodificar `SIN ESPECIFICAR` como NA. | Podría significar "no aplica" y no un dato faltante; por eso la fila no se elimina. |
| `STATUS` | Categorías poco frecuentes pero administrativamente válidas. | Normalizar texto; **no** fusionar categorías raras. | Agruparlas perdería información administrativa real. |
| `MODALIDAD` | Dominio limpio. | Normalizar texto y validar categorías. | Ninguno. |
| `JORNADA` | Categorías raras sin verificar si son reales. | Cruzar con `PLAN` antes de decidir cualquier recodificación. | Podría borrarse información válida de planes a distancia o de fin de semana. |
| `PLAN` | Categorías poco frecuentes. | Documentar sin fusionar. | Fusionar perdería granularidad relevante para el análisis. |
| `DEPARTAMENTAL` | Subdivisión administrativa (Guatemala en 4 zonas, Quiché en 2). | Validar que siempre corresponda al `DEPARTAMENTO` del registro. | Ninguno; es una verificación, no una modificación. |

### Criterios generales que atraviesan todo el plan

1. **No imputar.** Ningún valor faltante se rellena con una estimación. Un NA visible es preferible
   a un valor inventado que el analista no puede distinguir de un dato real.
2. **Marcar antes que borrar.** Cuando una decisión es discutible, se agrega una columna que la
   señala en lugar de aplicar el cambio en silencio.
3. **No unificar sin evidencia.** Las categorías poco frecuentes se documentan; solo se fusionan
   cuando se comprueba que representan el mismo valor escrito de forma distinta.

---

## 3. Variables originales (crudo del MINEDUC)

### `codigo_establecimiento`
- **Descripción:** identificador único del establecimiento asignado por el MINEDUC.
- **Tipo de dato:** `string` (se conserva como texto para no perder ceros a la izquierda ni los guiones).
- **Dominio permitido:** patrón `NN-NN-NNNN-NN` (departamento-distrito-secuencia-verificador).
- **Valores posibles:** cualquier código que cumpla el patrón; funciona como llave primaria.
- **Tratamiento aplicado:** se valida el formato (ver `codigo_establecimiento_valido`); **no se modifica ningún valor**. Las filas 100 % vacías del crudo se eliminaron antes de esta validación.
- **Nombre original en el crudo:** `CODIGO`.

### `codigo_distrito`
- **Descripción:** identificador del distrito educativo al que pertenece el establecimiento.
- **Tipo de dato:** `string`.
- **Dominio permitido:** dos formatos coexistentes en la fuente: `NN-NNN` o `NN-NN-NNNN`.
- **Valores posibles:** códigos en cualquiera de los dos formatos; ver `codigo_distrito_formato`.
- **Tratamiento aplicado:** se documenta el formato de cada valor en una variable derivada; **no se fuerza una conversión entre formatos** porque no existe regla segura para hacerlo sin inventar dígitos. Los valores truncados (sin dígitos después del último guion) se pasan a NA.
- **Nombre original en el crudo:** `DISTRITO`.

### `departamento`
- **Descripción:** departamento donde se ubica el establecimiento.
- **Tipo de dato:** `category`.
- **Dominio permitido:** los 22 departamentos oficiales de Guatemala.
- **Valores posibles:** GUATEMALA, EL PROGRESO, SACATEPEQUEZ, CHIMALTENANGO, ESCUINTLA, SANTA ROSA, SOLOLA, TOTONICAPAN, QUETZALTENANGO, SUCHITEPEQUEZ, RETALHULEU, SAN MARCOS, HUEHUETENANGO, QUICHE, BAJA VERAPAZ, ALTA VERAPAZ, PETEN, IZABAL, ZACAPA, CHIQUIMULA, JALAPA, JUTIAPA.
- **Tratamiento aplicado:** normalización de texto y mayúsculas; validado contra el catálogo oficial (0 valores fuera de catálogo).
- **Nombre original en el crudo:** `DEPARTAMENTO`.

### `municipio`
- **Descripción:** municipio donde se ubica el establecimiento.
- **Tipo de dato:** `category`.
- **Dominio permitido:** municipios oficiales de Guatemala (340 en total; el conjunto cubre `____`, ver limitación 1).
- **Valores posibles:** nombre del municipio en mayúsculas.
- **Tratamiento aplicado:** mayúsculas + detección de variantes de escritura por similitud de cadenas (RapidFuzz) **dentro de cada departamento**, para no fusionar municipios homónimos de departamentos distintos.
- **Nombre original en el crudo:** `MUNICIPIO`.

### `nombre_establecimiento`
- **Descripción:** nombre oficial o comercial del establecimiento.
- **Tipo de dato:** `string` (texto libre).
- **Dominio permitido:** cualquier texto.
- **Valores posibles:** texto libre en mayúsculas.
- **Tratamiento aplicado:** normalización general de texto; comillas tipográficas y simples unificadas a comillas dobles; comillas sueltas al inicio o al final eliminadas.
- **Nombre original en el crudo:** `ESTABLECIMIENTO`.

### `direccion`
- **Descripción:** dirección física del establecimiento.
- **Tipo de dato:** `string` (texto libre).
- **Dominio permitido:** cualquier texto.
- **Valores posibles:** texto libre en mayúsculas.
- **Tratamiento aplicado:** colapso de espacios múltiples, eliminación de espacios extremos, mayúsculas (lo que además unifica la abreviatura de ordinal: `8a.` → `8A.`) y eliminación de espacios antes de signos de puntuación.
- **Nombre original en el crudo:** `DIRECCION`.

### `telefono`
- **Descripción:** primer número telefónico válido (8 dígitos) encontrado en el campo original.
- **Tipo de dato:** `string` de 8 dígitos, o NA.
- **Dominio permitido:** exactamente 8 dígitos numéricos (formato de teléfono en Guatemala).
- **Valores posibles:** cadenas de 8 dígitos.
- **Tratamiento aplicado:** se extraen todas las secuencias de dígitos de la celda original (que podía traer varios números separados por guion, coma o texto) y se conservan las de 8 dígitos: la primera aquí, la segunda en `telefono_2`. Si ninguna califica, queda NA — **no se imputa ni se rellena**.
- **Nombre original en el crudo:** `TELEFONO`.

### `supervisor`
- **Descripción:** nombre del supervisor educativo asignado.
- **Tipo de dato:** `string` (texto libre).
- **Dominio permitido:** cualquier texto.
- **Valores posibles:** nombre de persona en mayúsculas.
- **Tratamiento aplicado:** normalización general de texto; placeholders de nulo convertidos a NA. **No se fusionan nombres similares**, porque personas distintas pueden tener nombres parecidos.
- **Nombre original en el crudo:** `SUPERVISOR`.

### `director`
- **Descripción:** nombre del director del establecimiento.
- **Tipo de dato:** `string` (texto libre).
- **Dominio permitido:** cualquier texto.
- **Valores posibles:** nombre de persona en mayúsculas.
- **Tratamiento aplicado:** igual que `supervisor`. **Importante para el analista:** el faltante puede representar una vacante real, no necesariamente un error de captura, por lo que no debe interpretarse como dato perdido.
- **Nombre original en el crudo:** `DIRECTOR`.

### `nivel`
- **Descripción:** nivel escolar del establecimiento. Constante en este conjunto por el filtro aplicado en la actividad 1.
- **Tipo de dato:** `category`.
- **Dominio permitido:** `DIVERSIFICADO`.
- **Valores posibles:** `DIVERSIFICADO` (único valor).
- **Tratamiento aplicado:** se conserva como control del filtro; una prueba de validación verifica que no haya otros niveles.
- **Nombre original en el crudo:** `NIVEL`.

### `sector`
- **Descripción:** sector administrativo del establecimiento.
- **Tipo de dato:** `category`.
- **Dominio permitido:** categorías administrativas observadas en la fuente.
- **Valores posibles:** *(verificar contra la corrida final)* OFICIAL, PRIVADO, COOPERATIVA, MUNICIPAL.
- **Tratamiento aplicado:** normalización de texto; categorías validadas sin fusionar.
- **Nombre original en el crudo:** `SECTOR`.

### `area`
- **Descripción:** área geográfica del establecimiento.
- **Tipo de dato:** `category`.
- **Dominio permitido:** {URBANA, RURAL} o NA.
- **Valores posibles:** URBANA, RURAL.
- **Tratamiento aplicado:** la categoría `SIN ESPECIFICAR` estaba fuera del dominio binario y se recodificó a NA. La fila **no se elimina**, de modo que el registro sigue siendo recuperable si el MINEDUC usara esa etiqueta con el sentido de "no aplica".
- **Nombre original en el crudo:** `AREA`.

### `estado`
- **Descripción:** estado administrativo/operativo del establecimiento.
- **Tipo de dato:** `category`.
- **Dominio permitido:** categorías administrativas observadas en la fuente.
- **Valores posibles:** *(verificar)* ABIERTA, CERRADA DEFINITIVAMENTE, CERRADA TEMPORALMENTE, TEMPORAL NOMBRAMIENTO, TEMPORAL TITULOS.
- **Tratamiento aplicado:** solo normalización de texto. Las categorías poco frecuentes se documentan pero **no se fusionan**: son estados administrativos reales y agruparlos perdería información.
- **Nombre original en el crudo:** `STATUS`.

### `modalidad`
- **Descripción:** modalidad lingüística de la oferta educativa.
- **Tipo de dato:** `category`.
- **Dominio permitido:** categorías observadas en la fuente.
- **Valores posibles:** MONOLINGUE, BILINGUE.
- **Tratamiento aplicado:** normalización de texto.
- **Nombre original en el crudo:** `MODALIDAD`.

### `jornada`
- **Descripción:** jornada en la que opera el establecimiento.
- **Tipo de dato:** `category`.
- **Dominio permitido:** categorías observadas en la fuente.
- **Valores posibles:** *(verificar)* MATUTINA, VESPERTINA, INTERMEDIA, NOCTURNA, DOBLE, FIN DE SEMANA.
- **Tratamiento aplicado:** normalización de texto. Las categorías poco frecuentes se cruzaron con `plan_educativo` (actividad 5h) para confirmar que corresponden a planes válidos antes de considerarlas un error; **no se recodificó nada sin evidencia**.
- **Nombre original en el crudo:** `JORNADA`.

### `plan_educativo`
- **Descripción:** plan educativo bajo el cual opera el establecimiento.
- **Tipo de dato:** `category`.
- **Dominio permitido:** categorías observadas en la fuente.
- **Valores posibles:** *(verificar)* DIARIO(REGULAR), FIN DE SEMANA, A DISTANCIA, entre otras.
- **Tratamiento aplicado:** normalización de texto; categorías raras documentadas y no fusionadas para no perder granularidad.
- **Nombre original en el crudo:** `PLAN`.

### `direccion_departamental`
- **Descripción:** dirección departamental de educación a la que reporta el establecimiento. Subdivide GUATEMALA en 4 zonas (Norte, Sur, Oriente, Occidente) y QUICHE en 2.
- **Tipo de dato:** `category`.
- **Dominio permitido:** nombre del departamento, con subdivisión donde aplica; siempre consistente con `departamento`.
- **Valores posibles:** *(verificar)* 26 valores observados.
- **Tratamiento aplicado:** normalización de texto; se validó el cruce con `departamento` (actividad 5h) sin encontrar contradicciones.
- **Nombre original en el crudo:** `DEPARTAMENTAL`.

---

## 4. Variables derivadas

Ninguna variable derivada reemplaza a una original: todas son **metadatos de calidad** que permiten
al analista filtrar según el nivel de confianza que necesite, sin repetir el proceso de limpieza.

| Variable | Tipo | Cómo se calculó | Utilidad |
|---|---|---|---|
| `departamento_municipio` | `string` | Concatenación `departamento + " / " + municipio`. | Llave geográfica única. Evita que se sumen municipios homónimos de departamentos distintos al agrupar. |
| `codigo_establecimiento_valido` | `bool` | `True` si `codigo_establecimiento` cumple el patrón `NN-NN-NNNN-NN`. | Auditar la integridad de la llave primaria sin alterar el valor original. |
| `codigo_distrito_formato` | `category` (`NN-NNN` / `NN-NN-NNNN` / NA) | Clasifica cada `codigo_distrito` según el patrón que sigue. | Documenta la coexistencia de dos convenciones sin forzar una conversión insegura. |
| `telefono_2` | `string` (8 dígitos) o NA | Segundo número de 8 dígitos hallado en la celda `TELEFONO` original. | Conserva el segundo contacto en lugar de descartarlo. |
| `telefono_valido` | `bool` | `True` si `telefono` no es NA. | Filtrar rápidamente los establecimientos contactables. |
| `telefono_multiple` | `bool` | `True` si la celda original contenía más de una secuencia de dígitos. | Trazabilidad: señala qué registros hubo que desagregar. |
| `posible_duplicado` | `bool` | `True` si el registro quedó agrupado con otro porque `nombre_establecimiento` **y** `direccion` superan sus umbrales de similitud (RapidFuzz `token_sort_ratio`, 90 y 85), calculado dentro de cada par (`departamento`, `municipio`). | Marca candidatos a duplicado parcial para revisión manual, sin eliminar registros. |
| `grupo_posible_duplicado` | `Int64` o NA | Identificador consecutivo de cada grupo de registros similares. | Permite inspeccionar juntos todos los registros de un mismo grupo. |
| `tipo_posible_duplicado` | `category` (`posible_duplicado_real` / `misma_sede_oferta_distinta` / NA) | Para cada grupo, compara `jornada`, `plan_educativo` y `sector`: si son idénticos en todo el grupo → `posible_duplicado_real`; si difieren → `misma_sede_oferta_distinta`. | Separa la granularidad legítima de la fuente de un duplicado real, y prioriza qué grupos revisar. |

---

## 5. Observaciones de los integrantes

> Esta sección documenta, para cada integrante, qué observó en los datos, qué decisiones tomó y
> por qué considera que la limpieza aplicada fue la adecuada. Es el complemento cualitativo del
> registro de transformaciones (actividad 6), que solo deja constancia del *qué*, no del *por qué*.

### 5.1 Observaciones — Integrante A · incisos 5a, 5b, 5c

*(Faltantes y placeholders · tipos de dato · normalización de texto)*

**Sobre los datos observados.**
`____` Describir aquí lo que se vio en el crudo: qué variables concentraban los faltantes, qué
formas de "dato ausente" aparecían disfrazadas de texto (`"-"`, `"."`, `"N/A"`, `"SIN DATO"`,
celdas con solo espacios), cuántas filas venían 100 % vacías y por qué distorsionaban los conteos.

**Decisiones tomadas.**
- `____` Por qué **no se imputó** ningún valor faltante.
- `____` Por qué el teléfono y los códigos se dejaron como texto y no como número.
- `____` Por qué se conservaron las tildes en el valor visible y solo se quitan en la clave de comparación.


`____`

**Por qué considero óptima esta limpieza.(Cindy)**

Mi criterio fue que una limpieza es buena cuando **hace visibles los problemas en lugar de
esconderlos**. Es fácil entregar un conjunto que se ve impecable: se borran las filas dudosas, se
imputan los faltantes y las métricas quedan perfectas. Pero ese conjunto miente, porque el analista
que lo reciba no tiene forma de saber qué se perdió en el camino.

Bajo ese criterio, la limpieza cumple en tres sentidos. Primero, **es reversible**: no se eliminó
ningún registro con información propia, y toda decisión discutible quedó marcada en una columna en
lugar de aplicada en silencio. Segundo, **es verificable**: las pruebas automáticas de la actividad
7 se ejecutan sobre el conjunto final y detienen el notebook si algo falla, así que el CSV limpio
nunca puede exportarse en un estado inválido. Tercero, **es honesta con sus límites**: las
decisiones que no pudimos tomar con evidencia —los duplicados parciales reales, las inconsistencias
entre variables, la brecha de municipios frente al catálogo del INE— están documentadas como
pendientes en lugar de resueltas a la fuerza.

La debilidad que reconozco es que los umbrales de similitud (90 y 85) son una elección de criterio,
no un óptimo demostrado. Los calibré revisando manualmente los grupos que producían, pero un
umbral distinto daría un conjunto de candidatos distinto. Por eso los dejo explícitos y
documentados aquí: cualquiera puede cambiarlos y volver a correr el notebook.

---
## 6. Notas de limitación y decisiones de diseño (acuerdos del grupo)

1. **`municipio` frente al catálogo oficial del INE.** No se validó exhaustivamente contra la lista
   completa de 340 municipios porque construirla de memoria conlleva riesgo de errores de
   transcripción que clasificarían como inválidos municipios reales. Se usó en su lugar la
   detección de variantes por similitud de cadenas (data-driven), que no encontró evidencia de
   errores de escritura. La brecha entre los municipios presentes y los 340 oficiales probablemente
   corresponde a municipios **sin ningún establecimiento de nivel Diversificado**. Queda pendiente
   de verificar contra la fuente oficial si se requiere una validación exhaustiva.
2. **`codigo_distrito`.** Los dos formatos observados se documentan pero no se convierten entre sí,
   por no existir una regla de mapeo verificable.
3. **Duplicados parciales.** Ningún registro se elimina automáticamente. Se entregan
   `posible_duplicado`, `grupo_posible_duplicado` y `tipo_posible_duplicado` para que el equipo o
   el analista revisen caso por caso.
4. **Valores faltantes.** No se imputó ningún valor. Un NA significa que el dato no estaba en la
   fuente o que era inválido; nunca un valor inventado. Como consecuencia, el conteo total de
   faltantes puede **subir** respecto al crudo: los placeholders de texto ya eran datos ausentes
   disfrazados, y convertirlos a NA hace el problema visible en lugar de esconderlo.
5. **Nombres de personas.** `supervisor` y `director` no se deduplican ni se corrigen por
   similitud: el riesgo de fusionar dos personas distintas con nombres parecidos supera el
   beneficio.

---

## 7. Notas de uso para el analista

1. Para un análisis conservador de duplicados, filtrar
   `tipo_posible_duplicado != "posible_duplicado_real"`. Los marcados como
   `misma_sede_oferta_distinta` **no son errores**.
2. Para agrupar geográficamente, usar `departamento_municipio` en lugar de `municipio` solo, o
   agrupar siempre por `departamento` **y** `municipio` a la vez.
3. Si `telefono_valido` es `False`, la celda original existía pero no contenía un número de 8
   dígitos: es un dato inválido en la fuente, no un dato ausente.
4. `nivel` es constante (`DIVERSIFICADO`) y se conserva únicamente como control del filtro aplicado.

---

## 8. Reproducibilidad

Ejecutar `Proyecto1.ipynb` de arriba hacia abajo. Las actividades 1 y 2 vuelven a descargar los
datos del sitio del MINEDUC si `establecimientos.csv` no existe; el resto del notebook parte de ese
archivo y regenera el CSV limpio. Todas las cifras citadas en este documento provienen de las
celdas del notebook, no de conteos hechos a mano.