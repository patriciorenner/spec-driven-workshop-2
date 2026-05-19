# TUC-002 — Detalle de una tabla del diccionario SAP

## Encabezado
- **ID:** TUC-002
- **Nombre:** Detalle de una tabla del diccionario SAP
- **UC de negocio asociado:** UC-001
- **Componente responsable:** módulo `detail` de la app Streamlit
- **Fecha:** [YYYY-MM-DD]
- **Estado:** `borrador`

## Contexto técnico

Al seleccionar una tabla en los resultados de búsqueda, la aplicación muestra los campos de esa tabla con su nombre, tipo de dato, longitud, descripción y data element asociado. La consulta se ejecuta sobre el mismo parquet, filtrando por `tabname`.

## Precondiciones técnicas

- Archivo `data/sap_dict.parquet` accesible.
- Esquema esperado: columnas `tabname`, `fieldname`, `datatype`, `leng`, `rollname`, `description` presentes.
- Nombre de tabla válido (no vacío, sin caracteres de control).

## Postcondiciones técnicas

- Detalle de la tabla visible en la UI.
- Sin escritura a disco. Sin mutación del dataset.

## Contrato de la interfaz

### Request

Función interna del módulo `detail`:

```
get_table_fields(tabname: str) -> pd.DataFrame
```

Parámetros:
- `tabname`: nombre exacto de la tabla, no vacío.

### Response exitosa

`pd.DataFrame` con columnas:
- `fieldname` (str)
- `datatype` (str)
- `leng` (int)
- `rollname` (str)
- `description` (str)

Ordenado por posición original del campo en la tabla.

### Responses de error

| Código | Condición | Body / mensaje |
|--------|-----------|----------------|
| `ValueError` | `tabname` vacío | "El nombre de tabla no puede estar vacío" |
| `LookupError` | `tabname` no existe en el dataset | "La tabla {tabname} no existe en el diccionario" |

## Diagrama de secuencia

```
[Usuario] -> [Streamlit UI] -> [get_table_fields] -> [DuckDB] -> [parquet]
    |             |                  |                  |           |
    |-- click --> |                  |                  |           |
    |             |-- tabname ------>|                  |           |
    |             |                  |-- SQL ---------->|           |
    |             |                  |                  |-- read -->|
    |             |                  |                  |<- rows ---|
    |             |                  |<-- DataFrame ----|           |
    |             |<-- DataFrame ----|                  |           |
    |<-- vista ---|                  |                  |           |
```

## Flujo principal

| Paso | Componente | Acción técnica |
|------|------------|----------------|
| 1 | Streamlit UI | Captura la selección de tabla desde resultados previos |
| 2 | módulo `detail` | Valida que `tabname` no esté vacío |
| 3 | módulo `detail` | Construye query DuckDB filtrando por `tabname` con parámetro bindeado |
| 4 | DuckDB | Ejecuta query y retorna filas |
| 5 | módulo `detail` | Verifica que el resultado no esté vacío |
| 6 | Streamlit UI | Renderiza el DataFrame como tabla expandida |

## Flujos alternativos

| ID | Condición | Descripción técnica |
|----|-----------|---------------------|
| A1 | Tabla sin campos | Se levanta `LookupError` y la UI muestra "tabla no encontrada" |
| A2 | Tabla con más de 1000 campos | Se trunca a 1000 filas con aviso en UI |

## Manejo de errores y excepciones

| ID | Condición | Comportamiento | Código de error |
|----|-----------|----------------|-----------------|
| E1 | `tabname` vacío | UI rechaza la acción | `ValueError` |
| E2 | Tabla inexistente | UI muestra mensaje informativo | `LookupError` |
| E3 | Parquet ausente | UI muestra mensaje crítico | `FileNotFoundError` |

## Requerimientos no funcionales

| Atributo | Valor objetivo | Notas |
|----------|----------------|-------|
| Latencia (p99) | < 500 ms | Lookup directo por `tabname` |
| Throughput | 1 usuario simultáneo | Alcance del workshop |
| Disponibilidad | N/A | Ejecución local |
| Timeout | 3 s | Query DuckDB |

## Seguridad

- **Autenticación:** no aplica.
- **Autorización:** no aplica.
- **Datos sensibles:** no aplica.
- **Inyección SQL:** parámetros bindeados; sin concatenación.
- **Rate limiting:** no aplica.

## Dependencias técnicas

| Componente / Servicio | Tipo | Versión mínima | Notas |
|-----------------------|------|----------------|-------|
| duckdb | Externo (PyPI) | 0.10 | |
| streamlit | Externo (PyPI) | 1.30 | |
| pandas | Externo (PyPI) | 2.0 | |

## ADRs relacionados

| ADR | Decisión relevante |
|-----|--------------------|
| ADR-001 | DuckDB como motor de consulta |
| ADR-002 | Streamlit como frontend |

## Criterios de aceptación técnicos

| ID | Criterio | Cómo verificar | Estado |
|----|----------|----------------|--------|
| TAC-1 | `get_table_fields` retorna todos los campos del `tabname` dado | Test unitario con tabla conocida (ej. `BKPF`) | `borrador` |
| TAC-2 | `get_table_fields` levanta `LookupError` si la tabla no existe | Test unitario con `tabname` inexistente | `borrador` |
| TAC-3 | `get_table_fields` levanta `ValueError` si `tabname` está vacío | Test unitario con `""` y `"   "` | `borrador` |
| TAC-4 | `get_table_fields` no es vulnerable a SQL injection | Test con `tabname="' OR 1=1 --"` | `borrador` |
| TAC-5 | `get_table_fields` responde en menos de 500 ms | Test de performance con `timeit` | `borrador` |

## Control de versiones

| Versión | Fecha | Autor | Cambios |
|---------|-------|-------|---------|
| 0.1 | | | Borrador inicial |
