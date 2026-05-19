# Stack Tecnológico — Explorador de Diccionario SAP

- **Proyecto:** Explorador de Diccionario SAP
- **Fecha:** [YYYY-MM-DD]

## Contexto

Aplicación web liviana para explorar el diccionario de datos de SAP (DDIC). El dataset está disponible como archivo parquet local y no requiere actualización en tiempo de ejecución. No hay usuarios concurrentes ni persistencia de datos del usuario; el alcance está acotado a consulta sobre el dataset.

## Stack

| Capa | Tecnología | Justificación |
|------|------------|---------------|
| Frontend | Streamlit 1.30+ | UI funcional en Python puro sin separación cliente/servidor; reduce drásticamente el código necesario y es bien conocido por Copilot. |
| Backend / API | — | No aplica. Streamlit ejecuta la lógica en el mismo proceso que la UI. |
| Base de datos | DuckDB 0.10+ sobre archivo parquet | Motor analítico embebido, lee parquet nativamente, sub-segundo sobre millones de filas, sin servidor. |
| Infraestructura | Ejecución local (`streamlit run`) | Alcance del workshop. Para producción se empaqueta como contenedor Docker. |
| Autenticación | No aplica | Datos no sensibles, ejecución local. |

## Integraciones externas

| Sistema | Protocolo | Notas |
|---------|-----------|-------|
| — | — | El sistema no integra con servicios externos. |

## ADRs asociados

| ADR | Decisión |
|-----|----------|
| ADR-001 | DuckDB como motor de consulta sobre parquet |
| ADR-002 | Streamlit como frontend web |
