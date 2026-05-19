# Plan de implementación: [nombre del plan]

<!-- ════════════════════════════════════════════════════
     PLAN — SOLO EL PLANNER ESCRIBE ESTAS SECCIONES (1–4)
     UNA VEZ APROBADO, ESTE BLOQUE ES INMUTABLE
     ════════════════════════════════════════════════════ -->

## 1. Encabezado

- **ID:** PLAN-[número]
- **Fecha de generación:** [YYYY-MM-DD]
- **Estado:** `borrador` <!-- borrador | aprobado -->
- **Aprobado por:** —
- **Fecha de aprobación:** —

## 2. Alcance

UCs y TUCs incluidos en este plan. Solo se planifican TUCs con `Estado: aprobado`.

| ID | Nombre | Tipo | TACs / ACs pendientes |
|----|--------|------|-----------------------|
|    |        | UC / TUC |                   |

## 3. Brechas

Especificaciones aprobadas excluidas del plan y su motivo.

| ID | Tipo | Motivo |
|----|------|--------|
|    | UC   | Sin TUC asociado |
|    | TUC  | Ambigüedad en TAC [ID] — [descripción del riesgo] |

## 4. Tareas

Cada fila es inmutable una vez aprobado el plan. El agente responsable solo escribe en la sección de seguimiento correspondiente.

| ID | Tipo | Agente | TUC | TAC(s) | Descripción | Depende de |
|----|------|--------|-----|--------|-------------|------------|
| T-001 | tests | tester | TUC-[n] | TAC-[n] | [qué tests escribir] | — |
| T-002 | código | coder | TUC-[n] | TAC-[n] | [qué implementar] | T-001 |
| T-003 | documentación | documenter | — | — | Actualizar README al cierre del plan | T-002 |

<!-- ════════════════════════════════════════════════════
     FIN DEL PLAN INMUTABLE
     ════════════════════════════════════════════════════ -->

---

<!-- ════════════════════════════════════════════════════
     ZONA DE SEGUIMIENTO — SOLO AGENTES ESCRIBEN AQUÍ
     No modificar los encabezados ni agregar secciones nuevas
     ════════════════════════════════════════════════════ -->

## 5. Seguimiento

Los agentes registran el avance en la sección de la tarea que tienen asignada. Solo se modifica el estado, las fechas, las notas y los bloqueantes.

### Progreso global

| Métrica | Valor |
|---------|-------|
| Total tareas | — |
| Pendientes | — |
| En proceso | — |
| Bloqueadas | — |
| Completas | — |
| Avance | — % |

> El agente que completa la última tarea actualiza esta tabla.

---

### T-001 — [descripción breve]

- **Estado:** `pendiente` <!-- pendiente | en proceso | bloqueado | completo -->
- **Agente:** —
- **Inicio:** —
- **Fin:** —
- **Notas:** —
- **Bloqueantes:** —

---

### T-002 — [descripción breve]

- **Estado:** `pendiente`
- **Agente:** —
- **Inicio:** —
- **Fin:** —
- **Notas:** —
- **Bloqueantes:** —

---

### T-003 — [descripción breve]

- **Estado:** `pendiente`
- **Agente:** —
- **Inicio:** —
- **Fin:** —
- **Notas:** —
- **Bloqueantes:** —
