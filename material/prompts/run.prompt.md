---
mode: agent
description: Ejecuta el plan aprobado end-to-end delegando en agentes especializados
tools:
  - read_file
  - create_file
  - list_dir
  - run_in_terminal
  - search_files
---

Ejecuta el plan aprobado end-to-end. Opera de forma autónoma: no solicites confirmación al usuario salvo que encuentres un bloqueante que requiera una decisión para continuar.

## Paso 1 — Cargar el plan

Lee los archivos en `specs/plans/` y selecciona el plan con `Estado: aprobado` que tenga tareas sin completar. Si hay más de uno, pregunta al usuario cuál ejecutar antes de continuar. Si no hay ninguno, informa y detente.

Carga el plan completo: encabezado, alcance, tareas y secciones de seguimiento.

## Paso 2 — Ciclo de ejecución

Repite hasta que todas las tareas estén en estado `completo` o encuentres un bloqueante:

### 2a. Seleccionar la próxima tarea

Identifica todas las tareas con `Estado: pendiente` cuyas dependencias (`Depende de`) estén en estado `completo` o sean `—`. Ejecuta la de menor ID disponible.

### 2b. Iniciar la tarea

En la sección de seguimiento de la tarea seleccionada, actualiza:
- `Estado` → `en proceso`
- `Agente` → nombre del agente responsable
- `Inicio` → fecha y hora actual

### 2c. Delegar en el agente especializado

Según el tipo de la tarea, invoca el agente correspondiente pasándole el contexto mínimo necesario: ID de tarea, TUC de referencia, TACs cubiertos, y path del plan para que pueda registrar bloqueantes si los encuentra.

| Tipo | Agente | Definición |
|------|--------|------------|
| tests | tester | [TESTER.md](material/agents/TESTER.md) |
| código | coder | [CODER.md](material/agents/CODER.md) |
| documentación | documenter | [DOCUMENTER.md](material/agents/DOCUMENTER.md) |

### 2d. Registrar el resultado

**Si la tarea se completó:** actualiza la sección de seguimiento:
- `Estado` → `completo`
- `Fin` → fecha y hora actual
- `Notas` → resumen de una línea del trabajo realizado

**Si la tarea falló y el agente puede reintentar:** registra el intento en `Notas` y vuelve al paso 2c. Máximo tres intentos por tarea.

**Si la tarea está bloqueada** (tres intentos fallidos o el agente reporta un bloqueante explícito): actualiza la sección de seguimiento:
- `Estado` → `bloqueado`
- `Bloqueantes` → descripción del problema

Luego escala al usuario con este formato y detente:

---
**Bloqueante en [T-XXX] — [descripción breve de la tarea]**

- **Qué se intentó:** [resumen de los intentos]
- **Qué falló:** [causa raíz identificada]
- **Decisión requerida:** [pregunta concreta al usuario]
---

### 2e. Actualizar progreso global

Después de cada tarea completada, recalcula y actualiza la tabla de Progreso global en la sección 5 del plan.

## Paso 3 — Cierre

Cuando todas las tareas estén en estado `completo`:

1. Actualiza la tabla de Progreso global: Avance → 100%.
2. Actualiza el estado de cada TAC referenciado en el plan a `completo` en su TUC correspondiente.
3. Actualiza el estado global de cada TUC cubierto por el plan a `completo`.
4. Notifica al usuario: plan completado, lista de artefactos generados o modificados.

## Restricciones

- No modifica las secciones 1–4 del plan (son inmutables); solo escribe en las secciones de seguimiento.
- No toma decisiones de arquitectura, negocio ni diseño de tests; las escala como bloqueantes.
- No omite tareas ni reordena la secuencia definida en el plan.
- Si una tarea completada genera hallazgos que requieren nuevas tareas fuera del plan actual, los reporta al usuario al cierre — no agrega tareas al plan en curso.
