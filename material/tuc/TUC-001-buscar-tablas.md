# TUC-001 — Buscar tablas del diccionario SAP

## Encabezado
- **ID:** TUC-001
- **Nombre:** Buscar tablas del diccionario SAP
- **UC de negocio asociado:** UC-001
- **Componente responsable:** módulo `search` de la app Streamlit
- **Fecha:** [YYYY-MM-DD]
- **Estado:** `borrador`

## Contexto técnico

El usuario ingresa un término en la interfaz Streamlit. La aplicación consulta el archivo parquet del diccionario SAP usando DuckDB y retorna las tablas cuyo nombre o descripción contienen el término. La consulta se ejecuta in-process; no hay capa de red ni almacenamiento intermedio.

## Precondiciones técnicas

- Archivo `data/sap_dict.parquet` accesible en disco.
- Esquema esperado: columnas `tabname`, `tabclass`, `description` presentes.
- DuckDB disponible como dependencia de Python.

## Postcondiciones técnicas

- Resultados visibles en la UI.
- Sin escritura a disco. Sin mutación del dataset.

## Contrato de la interfaz

### Request

Función interna del módulo `search`:

```
search_tables(term: str, limit: int = 100) -> pd.DataFrame
```

Parámetros:
- `term`: texto de búsqueda, no vacío después de `strip()`.
- `limit`: máximo de filas retornadas; por defecto 100.

### Response exitosa

`pd.DataFrame` con columnas:
- `tabname` (str)
- `tabclass` (str)
- `description` (str)
- `match_score` (int): cantidad de coincidencias del término en nombre + descripción.

Ordenado por `match_score` descendente.

### Responses de error

| Código | Condición | Body / mensaje |
|--------|-----------|----------------|
| `ValueError` | `term` vacío después de `strip()` | "El término de búsqueda no puede estar vacío" |
| `FileNotFoundError` | parquet ausente | "Dataset no encontrado en data/sap_dict.parquet" |

## Diagrama de secuencia

```
[Usuario] -> [Streamlit UI] -> [search_tables] -> [DuckDB] -> [parquet]
    |             |                 |                |           |
    |-- término ->|                 |                |           |
    |             |-- search ------>|                |           |
    |             |                 |-- SQL -------->|           |
    |             |                 |                |-- read -->|
    |             |                 |                |<- rows ---|
    |             |                 |<-- DataFrame --|           |
    |             |<-- DataFrame ---|                |           |
    |<-- tabla ---|                 |                |           |
```

## Flujo principal

| Paso | Componente | Acción técnica |
|------|------------|----------------|
| 1 | Streamlit UI | Recibe input de texto del usuario |
| 2 | Streamlit UI | Valida que el término no esté vacío |
| 3 | módulo `search` | Construye query DuckDB con `ILIKE` sobre `tabname` y `description` |
| 4 | DuckDB | Ejecuta query sobre parquet y retorna filas |
| 5 | módulo `search` | Calcula `match_score` y ordena |
| 6 | Streamlit UI | Renderiza el DataFrame como tabla |

## Flujos alternativos

| ID | Condición | Descripción técnica |
|----|-----------|---------------------|
| A1 | Término sin resultados | Se retorna DataFrame vacío; la UI muestra "sin resultados" |
| A2 | Término con caracteres especiales SQL | Se escapan vía parámetros bindeados de DuckDB |

## Manejo de errores y excepciones

| ID | Condición | Comportamiento | Código de error |
|----|-----------|----------------|-----------------|
| E1 | Parquet inexistente | UI muestra mensaje y termina con código 1 | `FileNotFoundError` |
| E2 | Schema inesperado en parquet | UI muestra "schema inválido" | `ValueError` |
| E3 | Término vacío después de `strip()` | UI deshabilita el botón de búsqueda | UI-level |

## Requerimientos no funcionales

| Atributo | Valor objetivo | Notas |
|----------|----------------|-------|
| Latencia (p99) | < 1 s | Sobre dataset de hasta 100k filas |
| Throughput | 1 usuario simultáneo | Alcance del workshop |
| Disponibilidad | N/A | Ejecución local |
| Timeout | 5 s | Query DuckDB cancelada si excede |

## Seguridad

- **Autenticación:** no aplica (ejecución local).
- **Autorización:** no aplica.
- **Datos sensibles:** el diccionario SAP no contiene datos productivos.
- **Inyección SQL:** queries con parámetros bindeados; nunca concatenación de strings.
- **Rate limiting:** no aplica (entorno local).

## Dependencias técnicas

| Componente / Servicio | Tipo | Versión mínima | Notas |
|-----------------------|------|----------------|-------|
| duckdb | Externo (PyPI) | 0.10 | Motor de consulta |
| streamlit | Externo (PyPI) | 1.30 | Framework UI |
| pandas | Externo (PyPI) | 2.0 | Manipulación de DataFrame |

## ADRs relacionados

| ADR | Decisión relevante |
|-----|--------------------|
| ADR-001 | Uso de DuckDB como motor de consulta |

## Criterios de aceptación técnicos

| ID | Criterio | Cómo verificar | Estado |
|----|----------|----------------|--------|
| TAC-1 | `search_tables` retorna las filas cuyo `tabname` o `description` contiene el término (case-insensitive) | Test unitario con dataset de prueba | `borrador` |
| TAC-2 | `search_tables` retorna como máximo `limit` filas | Test unitario con `limit=5` sobre dataset con >5 matches | `borrador` |
| TAC-3 | `search_tables` levanta `ValueError` si `term` queda vacío tras `strip()` | Test unitario con `term=""` y `term="   "` | `borrador` |
| TAC-4 | `search_tables` responde en menos de 1 s sobre dataset de 100k filas | Test de performance con `timeit` | `borrador` |
| TAC-5 | `search_tables` no es vulnerable a SQL injection | Test con `term="' OR 1=1 --"` verifica que no retorna todas las filas | `borrador` |

## Control de versiones

| Versión | Fecha | Autor | Cambios |
|---------|-------|-------|---------|
| 0.1 | | | Borrador inicial |
