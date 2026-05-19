---
mode: agent
description: Orquesta la revisión de código, planifica correcciones y coordina agentes hasta resolver todos los hallazgos críticos y mayores
tools:
  - read_file
  - create_file
  - list_dir
  - run_in_terminal
  - search_files
---

Ejecuta el ciclo de revisión de código sobre el alcance indicado. Opera de forma autónoma: no solicites confirmación al usuario salvo que encuentres un bloqueante que requiera una decisión para continuar.

El alcance se expresa como una lista de archivos, un módulo o un conjunto de cambios. Si no se indica, solicítalo al usuario antes de continuar.

## Paso 1 — Auditoría inicial

Invoca al agente `code-reviewer` pasándole el alcance completo.

El agente produce un informe con hallazgos clasificados por severidad: Crítico, Alto, Medio y Bajo. Guarda el informe como variable de trabajo; no lo guardes en disco.

Si el informe no contiene hallazgos Críticos ni Altos, notifica al usuario con el resumen del informe y detente: no hay correcciones que planificar.

## Paso 2 — Planificación de correcciones

A partir del informe, construye un plan de corrección que incluya únicamente los hallazgos Críticos y Altos.

Para cada hallazgo, determina el agente responsable según su naturaleza:

| Naturaleza del hallazgo | Agente responsable |
|-------------------------|-------------------|
| Código de producción (lógica, seguridad, calidad) | coder |
| Tests faltantes o incorrectos | tester |
| Documentación desactualizada o faltante | documenter |
| Decisión de arquitectura requerida | architect |

Agrupa los hallazgos por agente. Si un hallazgo requiere coordinación entre dos agentes (por ejemplo, coder y tester), registra la dependencia: el tester actúa primero, el coder implementa contra los tests.

Presenta el plan al usuario en este formato antes de ejecutarlo:

---
**Plan de corrección — Iteración 1**

| # | Hallazgo | Severidad | Agente | Depende de |
|---|----------|-----------|--------|------------|
| … | … | … | … | … |
---

Espera confirmación explícita del usuario antes de continuar.

## Paso 3 — Ciclo de corrección y verificación

Repite hasta que no queden hallazgos Críticos ni Altos sin solución confirmada, o hasta completar 3 iteraciones.

### 3a. Delegar correcciones

Para cada grupo de hallazgos del plan, invoca el agente responsable pasándole:
- La lista de hallazgos asignados (archivo, línea, descripción, recomendación).
- El alcance original para que pueda ubicar contexto.
- Las restricciones del agente según su definición en `material/agents/`.

Respeta las dependencias definidas en el plan: no delegues en el coder hasta que el tester haya completado los tests asociados.

### 3b. Re-auditar hallazgos sin solución confirmada

Una vez que los agentes reportan sus correcciones, invoca al agente `code-reviewer` nuevamente. El alcance de esta auditoría se limita a:
- Los archivos modificados en esta iteración.
- Los hallazgos de iteraciones anteriores que aún no tienen solución confirmada.

Considera un hallazgo resuelto únicamente si el informe de re-auditoría no lo vuelve a reportar.

### 3c. Evaluar resultado de la iteración

**Si no quedan hallazgos Críticos ni Altos sin solución:** avanza al Paso 4.

**Si quedan hallazgos sin solución y no se alcanzaron 3 iteraciones:** registra los hallazgos pendientes, incrementa el contador de iteración y vuelve al paso 3a con el alcance reducido a esos hallazgos.

**Si se completaron 3 iteraciones y aún quedan hallazgos sin solución:** avanza al Paso 4 con estado de escalada.

## Paso 4 — Cierre

**Si todos los hallazgos Críticos y Altos están resueltos:**

Notifica al usuario con este resumen:
- Hallazgos resueltos por severidad y agente.
- Hallazgos Medios y Bajos pendientes (sin plan de corrección; quedan a criterio del coder).
- Archivos modificados durante el proceso.

**Si quedan hallazgos sin resolver después de 3 iteraciones:**

Escala al usuario con este formato y detente:

---
**Revisión de código — Escalada tras 3 iteraciones**

Hallazgos sin solución confirmada:

| # | Hallazgo | Severidad | Iteraciones intentadas | Causa raíz identificada |
|---|----------|-----------|------------------------|------------------------|
| … | … | … | … | … |

**Decisión requerida:** [descripción de la decisión o información necesaria para continuar]
---

## Restricciones

- No modifica código, tests ni documentación directamente; delega siempre en el agente especializado.
- No omite hallazgos Críticos ni Altos del plan de corrección.
- No amplía el alcance de la auditoría más allá del alcance original más los archivos modificados en las iteraciones.
- Si el agente `code-reviewer` introduce hallazgos nuevos no relacionados con el alcance original, los registra como observaciones al cierre pero no los incorpora al ciclo de corrección en curso.
- Si un agente especializado reporta un bloqueante durante una corrección, lo registra como causa raíz del hallazgo asociado y lo incluye en la escalada final.
