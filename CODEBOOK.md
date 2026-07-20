# Libro de Códigos — Establecimientos educativos de nivel Diversificado (Guatemala)

**Proyecto 1 — CC3084 Data Science, Universidad del Valle de Guatemala, Semestre II 2026**

| | |
|---|---|
| **Fuente de los datos** | Sistema de Búsqueda de Establecimientos Educativos, MINEDUC Guatemala — http://www.mineduc.gob.gt/BUSCAESTABLECIMIENTO_GE/ |
| **Fecha de extracción** | 2026-07-18 |
| **Versión del conjunto limpio** | 1.0 |
| **Archivo** | `data/clean/establecimientos_diversificado_limpio.csv` |
| **Registros** | 9,706 |
| **Variables** | 26 (17 originales + 9 derivadas) |
| **Cobertura** | los 22 departamentos del país, nivel escolar DIVERSIFICADO |
| **Codificación** | UTF-8 con BOM (utf-8-sig) |

---

## Variables

### `codigo_establecimiento`

- **Descripción:** Código único del establecimiento asignado por el MINEDUC.
- **Tipo de dato:** `string`
- **Dominio permitido:** Formato NN-NN-NNNN-NN (departamento-distrito-correlativo-verificador).
- **Valores posibles:** 9706 valores distintos (ej.: `16-14-0107-46`, `16-14-1344-46`, `16-14-0112-46`, ...)
- **Valores faltantes:** 0 (0.00%)
- **Tratamiento aplicado durante la limpieza:** Validado por expresión regular; no se modificó ningún valor.

### `codigo_distrito`

- **Descripción:** Código del distrito escolar al que pertenece el establecimiento.
- **Tipo de dato:** `string`
- **Dominio permitido:** Formato NN-NNN o NN-NN-NNNN (coexisten dos convenciones en la fuente).
- **Valores posibles:** 1590 valores distintos (ej.: `05-033`, `01-411`, `05-007`, ...)
- **Valores faltantes:** 317 (3.27%)
- **Tratamiento aplicado durante la limpieza:** Los códigos truncados se pasaron a NA; el formato se documenta en codigo_distrito_formato.

### `departamento`

- **Descripción:** Departamento de Guatemala donde se ubica el establecimiento.
- **Tipo de dato:** `category`
- **Dominio permitido:** Uno de los 22 departamentos oficiales.
- **Valores posibles:** 22 valores distintos (ej.: `GUATEMALA`, `SAN MARCOS`, `ESCUINTLA`, ...)
- **Valores faltantes:** 0 (0.00%)
- **Tratamiento aplicado durante la limpieza:** Normalizado a mayúsculas y validado contra el catálogo oficial.

### `municipio`

- **Descripción:** Municipio donde se ubica el establecimiento.
- **Tipo de dato:** `category`
- **Dominio permitido:** Municipio perteneciente al departamento del registro.
- **Valores posibles:** 330 valores distintos (ej.: `MIXCO`, `VILLA NUEVA`, `QUETZALTENANGO`, ...)
- **Valores faltantes:** 0 (0.00%)
- **Tratamiento aplicado durante la limpieza:** Normalizado a mayúsculas; variantes de escritura unificadas por similitud dentro de cada departamento.

### `departamento_municipio`  *(variable derivada)*

- **Descripción:** Llave geográfica 'DEPARTAMENTO / MUNICIPIO'.
- **Tipo de dato:** `string`
- **Dominio permitido:** Concatenación de departamento y municipio.
- **Valores posibles:** 336 valores distintos (ej.: `GUATEMALA / MIXCO`, `GUATEMALA / VILLA NUEVA`, `QUETZALTENANGO / QUETZALTENANGO`, ...)
- **Valores faltantes:** 0 (0.00%)
- **Tratamiento aplicado durante la limpieza:** Variable derivada: desambigua municipios homónimos al agrupar.

### `direccion_departamental`

- **Descripción:** Dirección departamental del MINEDUC responsable del establecimiento.
- **Tipo de dato:** `category`
- **Dominio permitido:** Nombre del departamento, con subdivisión en Guatemala y Quiché.
- **Valores posibles:** 26 valores distintos (ej.: `GUATEMALA SUR`, `GUATEMALA OCCIDENTE`, `SAN MARCOS`, ...)
- **Valores faltantes:** 0 (0.00%)
- **Tratamiento aplicado durante la limpieza:** Normalizada; validada contra departamento (5h).

### `nombre_establecimiento`

- **Descripción:** Nombre oficial del establecimiento educativo.
- **Tipo de dato:** `string`
- **Dominio permitido:** Texto libre en mayúsculas.
- **Valores posibles:** 5035 valores distintos (ej.: `INSTITUTO NACIONAL DE EDUCACION DIVERSIFICADA`, `INED INSTITUTO NACIONAL DE EDUCACIÓN DIVERSIFICADA`, `INSTITUTO NACIONAL DE EDUCACIÓN DIVERSIFICADA`, ...)
- **Valores faltantes:** 3 (0.03%)
- **Tratamiento aplicado durante la limpieza:** Normalización de texto; comillas unificadas a comillas dobles.

### `direccion`

- **Descripción:** Dirección física del establecimiento.
- **Tipo de dato:** `string`
- **Dominio permitido:** Texto libre en mayúsculas.
- **Valores posibles:** 6021 valores distintos (ej.: `CABECERA MUNICIPAL`, `BARRIO EL CENTRO`, `BARRIO EL CALVARIO`, ...)
- **Valores faltantes:** 70 (0.72%)
- **Tratamiento aplicado durante la limpieza:** Normalización de espacios, mayúsculas y puntuación.

### `telefono`

- **Descripción:** Primer número telefónico de contacto.
- **Tipo de dato:** `string`
- **Dominio permitido:** Exactamente 8 dígitos, o NA.
- **Valores posibles:** 5468 valores distintos (ej.: `79480009`, `77602663`, `45353648`, ...)
- **Valores faltantes:** 745 (7.68%)
- **Tratamiento aplicado durante la limpieza:** Extraído de la celda original; los valores que no son de 8 dígitos quedaron en NA.

### `telefono_2`  *(variable derivada)*

- **Descripción:** Segundo número telefónico, cuando la celda original traía más de uno.
- **Tipo de dato:** `string`
- **Dominio permitido:** Exactamente 8 dígitos, o NA.
- **Valores posibles:** 72 valores distintos (ej.: `24329098`, `79480166`, `79540909`, ...)
- **Valores faltantes:** 9,613 (99.04%)
- **Tratamiento aplicado durante la limpieza:** Variable derivada de la desagregación del campo telefónico original.

### `supervisor`

- **Descripción:** Nombre del supervisor educativo asignado.
- **Tipo de dato:** `string`
- **Dominio permitido:** Texto libre en mayúsculas.
- **Valores posibles:** 1241 valores distintos (ej.: `ELENA ELIZABETH SUCHITE GARNICA DE QUINTANILLA`, `REMY ARTURO SINAY GUDIEL`, `JUAN ENRIQUE MARTINEZ SOLANO`, ...)
- **Valores faltantes:** 304 (3.13%)
- **Tratamiento aplicado durante la limpieza:** Normalización de texto; placeholders de nulo convertidos a NA. No se fusionaron nombres similares.

### `director`

- **Descripción:** Nombre del director del establecimiento.
- **Tipo de dato:** `string`
- **Dominio permitido:** Texto libre en mayúsculas.
- **Valores posibles:** 4848 valores distintos (ej.: `---`, `.`, `MARÍA DOLORES PÉREZ TUCHÁN`, ...)
- **Valores faltantes:** 1,247 (12.85%)
- **Tratamiento aplicado durante la limpieza:** Igual que supervisor. El faltante puede corresponder a una vacante real, no a un error.

### `nivel`

- **Descripción:** Nivel escolar del establecimiento.
- **Tipo de dato:** `category`
- **Dominio permitido:** DIVERSIFICADO (constante, por el filtro de la actividad 1).
- **Valores posibles:** `DIVERSIFICADO`
- **Valores faltantes:** 0 (0.00%)
- **Tratamiento aplicado durante la limpieza:** Se conserva como control del filtro aplicado.

### `sector`

- **Descripción:** Sector administrativo del establecimiento.
- **Tipo de dato:** `category`
- **Dominio permitido:** Categorías observadas en la fuente (OFICIAL, PRIVADO, COOPERATIVA, MUNICIPAL).
- **Valores posibles:** `COOPERATIVA`, `MUNICIPAL`, `OFICIAL`, `PRIVADO`
- **Valores faltantes:** 0 (0.00%)
- **Tratamiento aplicado durante la limpieza:** Normalización de texto; categorías validadas sin fusionar.

### `area`

- **Descripción:** Área geográfica del establecimiento.
- **Tipo de dato:** `category`
- **Dominio permitido:** URBANA o RURAL, o NA.
- **Valores posibles:** `RURAL`, `URBANA`
- **Valores faltantes:** 2 (0.02%)
- **Tratamiento aplicado durante la limpieza:** 'SIN ESPECIFICAR' recodificado a NA por estar fuera del dominio binario.

### `estado`

- **Descripción:** Estado administrativo del establecimiento (STATUS en la fuente).
- **Tipo de dato:** `category`
- **Dominio permitido:** Categorías administrativas observadas en la fuente.
- **Valores posibles:** `ABIERTA`, `CERRADA DEFINITIVAMENTE`, `CERRADA TEMPORALMENTE`, `TEMPORAL NOMBRAMIENTO`, `TEMPORAL TITULOS`
- **Valores faltantes:** 0 (0.00%)
- **Tratamiento aplicado durante la limpieza:** Normalización de texto; categorías poco frecuentes documentadas, no fusionadas.

### `modalidad`

- **Descripción:** Modalidad educativa.
- **Tipo de dato:** `category`
- **Dominio permitido:** Categorías observadas en la fuente.
- **Valores posibles:** `BILINGUE`, `MONOLINGUE`
- **Valores faltantes:** 0 (0.00%)
- **Tratamiento aplicado durante la limpieza:** Normalización de texto.

### `jornada`

- **Descripción:** Jornada en que opera el establecimiento.
- **Tipo de dato:** `category`
- **Dominio permitido:** Categorías observadas en la fuente.
- **Valores posibles:** `DOBLE`, `INTERMEDIA`, `MATUTINA`, `NOCTURNA`, `SIN JORNADA`, `VESPERTINA`
- **Valores faltantes:** 0 (0.00%)
- **Tratamiento aplicado durante la limpieza:** Normalización de texto; cruzada con plan_educativo para validar categorías raras.

### `plan_educativo`

- **Descripción:** Plan educativo (PLAN en la fuente).
- **Tipo de dato:** `category`
- **Dominio permitido:** Categorías observadas en la fuente.
- **Valores posibles:** 13 valores distintos (ej.: `DIARIO(REGULAR)`, `FIN DE SEMANA`, `SEMIPRESENCIAL (FIN DE SEMANA)`, ...)
- **Valores faltantes:** 0 (0.00%)
- **Tratamiento aplicado durante la limpieza:** Normalización de texto.

### `codigo_establecimiento_valido`  *(variable derivada)*

- **Descripción:** Indica si el código cumple el formato NN-NN-NNNN-NN.
- **Tipo de dato:** `boolean`
- **Dominio permitido:** True / False.
- **Valores posibles:** `True`
- **Valores faltantes:** 0 (0.00%)
- **Tratamiento aplicado durante la limpieza:** Variable derivada: control de integridad de la llave primaria.

### `codigo_distrito_formato`  *(variable derivada)*

- **Descripción:** Convención de formato usada por codigo_distrito.
- **Tipo de dato:** `category`
- **Dominio permitido:** 'NN-NNN', 'NN-NN-NNNN' o NA.
- **Valores posibles:** `NN-NN-NNNN`, `NN-NNN`
- **Valores faltantes:** 317 (3.27%)
- **Tratamiento aplicado durante la limpieza:** Variable derivada: documenta el formato sin alterar el valor original.

### `telefono_valido`  *(variable derivada)*

- **Descripción:** Indica si el registro tiene al menos un teléfono de 8 dígitos.
- **Tipo de dato:** `bool`
- **Dominio permitido:** True / False.
- **Valores posibles:** `False`, `True`
- **Valores faltantes:** 0 (0.00%)
- **Tratamiento aplicado durante la limpieza:** Variable derivada: permite filtrar registros contactables.

### `telefono_multiple`  *(variable derivada)*

- **Descripción:** Indica si la celda original contenía más de un número.
- **Tipo de dato:** `bool`
- **Dominio permitido:** True / False.
- **Valores posibles:** `False`, `True`
- **Valores faltantes:** 0 (0.00%)
- **Tratamiento aplicado durante la limpieza:** Variable derivada: trazabilidad de la desagregación del campo.

### `posible_duplicado`  *(variable derivada)*

- **Descripción:** Indica si el registro pertenece a un grupo de similitud nombre+dirección.
- **Tipo de dato:** `bool`
- **Dominio permitido:** True / False.
- **Valores posibles:** `False`, `True`
- **Valores faltantes:** 0 (0.00%)
- **Tratamiento aplicado durante la limpieza:** Variable derivada (RapidFuzz token_sort_ratio dentro de cada departamento-municipio). No se eliminó ningún registro.

### `grupo_posible_duplicado`  *(variable derivada)*

- **Descripción:** Identificador del grupo de posibles duplicados.
- **Tipo de dato:** `Int64`
- **Dominio permitido:** Entero positivo o NA.
- **Valores posibles:** 1756 valores distintos (ej.: `607`, `610`, `1571`, ...)
- **Valores faltantes:** 4,705 (48.48%)
- **Tratamiento aplicado durante la limpieza:** Variable derivada: permite inspeccionar juntos los registros de un grupo.

### `tipo_posible_duplicado`  *(variable derivada)*

- **Descripción:** Clasificación del grupo de posibles duplicados.
- **Tipo de dato:** `category`
- **Dominio permitido:** 'misma_sede_oferta_distinta', 'posible_duplicado_real' o NA.
- **Valores posibles:** `misma_sede_oferta_distinta`, `posible_duplicado_real`
- **Valores faltantes:** 4,705 (48.48%)
- **Tratamiento aplicado durante la limpieza:** Variable derivada: distingue la granularidad legítima de la fuente de un duplicado real.

---

## Notas de uso

1. **Duplicados parciales:** ningún registro marcado como `posible_duplicado` fue eliminado.
   Para un análisis conservador, filtrar `tipo_posible_duplicado != 'posible_duplicado_real'`.
   Los marcados como `misma_sede_oferta_distinta` **no son errores**: son el mismo
   establecimiento con más de una jornada, plan o sector.
2. **Municipios homónimos:** para agrupar geográficamente, usar `departamento_municipio` en
   lugar de `municipio` solo.
3. **Faltantes:** no se imputó ningún valor. Un `NA` significa que el dato no estaba en la
   fuente o que era inválido; nunca un valor inventado.
4. **Teléfonos:** solo se conservan números de 8 dígitos. Si `telefono_valido` es `False`, la
   celda original existía pero no contenía un número válido.

## Reproducibilidad

Ejecutar `Proyecto1.ipynb` de arriba hacia abajo. Las actividades 1-2 vuelven a descargar los datos del
sitio del MINEDUC si `establecimientos.csv` no existe; el resto del notebook parte de ese
archivo y regenera tanto el CSV limpio como este libro de códigos.
