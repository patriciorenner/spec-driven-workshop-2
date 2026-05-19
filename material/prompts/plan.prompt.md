---
mode: agent
description: Genera el plan de implementación a partir de las especificaciones aprobadas
tools:
  - read_file
  - list_dir
  - create_file
---

Genera un plan de implementación a partir de las especificaciones aprobadas del proyecto.

## Paso 1 — Análisis de estado

Lee todos los archivos en `specs/uc/` y en `specs/tuc/`. Identifica:

- **UCs pendientes**: `Estado: aprobado` con al menos un AC en estado distinto de `completo`.
- **TUCs pendientes**: `Estado: aprobado` con al menos un TAC en estado distinto de `completo`.
- **Brechas**: UCs aprobados sin TUC asociado.

Presenta el resumen al usuario antes de continuar. Si no hay TUCs pendientes, informa al usuario y detente.

## Paso 2 — Descomposición en tareas

Para cada TUC con TACs pendientes, crea tareas en este orden:

1. **Tests** (tester) — una tarea por TAC o por grupo lógico de TACs del mismo TUC.
2. **Código** (coder) — una tarea por TUC; depende de las tareas de tests del mismo TUC.
3. **Documentación** (documenter) — una tarea global al final; depende de que todas las tareas de código estén completas.

Asigna IDs correlativos (`T-001`, `T-002`, …). Registra las dependencias entre tareas.

## Paso 3 — Redacción del plan

Usa el template en `material/templates/plan-template.md`. Completa:

- **Sección 1 — Encabezado**: asigna el próximo ID disponible en `specs/plans/` (PLAN-001 si no existe ninguno), fecha de hoy, estado `borrador`.
- **Sección 2 — Alcance**: lista los UCs y TUCs incluidos con el conteo de ACs/TACs pendientes.
- **Sección 3 — Brechas**: lista los UCs sin TUC y cualquier ambigüedad encontrada en las especificaciones.
- **Sección 4 — Tareas**: completa la tabla con las tareas generadas en el paso 2.
- **Sección 5 — Seguimiento**: genera una subsección por tarea con todos los campos en estado `pendiente`.

## Paso 4 — Aprobación

Presenta el plan completo al usuario y solicita aprobación explícita. No guardes el archivo hasta recibirla.

Al aprobar:
- Actualiza `Estado` a `aprobado` en la sección 1.
- Completa `Aprobado por` y `Fecha de aprobación`.
- Guarda el archivo en `specs/plans/PLAN-[número]-[slug].md`.

El plan aprobado es un contrato. Las secciones 1–4 no se modifican después de guardado.
