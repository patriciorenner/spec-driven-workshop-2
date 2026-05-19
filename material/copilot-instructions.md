# Copilot Instructions

Este proyecto sigue **Spec Driven Development**: las especificaciones son un contrato y tienen precedencia sobre tests y código. El desarrollo de código sigue **Test Driven Development** con ciclo rojo → verde → refactorización.

## Pipeline de desarrollo

**Fase interactiva** — el usuario orquesta y proporciona feedback en cada paso:

1. Especificaciones funcionales
2. Especificaciones técnicas
3. Planificación

**Fase autónoma** — un agente orquesta el flujo completo usando subagentes:

1. Tests — escribe los tests que fallan (rojo)
2. Código — implementa hasta que los tests pasen (verde), luego refactoriza
3. *(cuando todas las tareas de código están completas)* Documentación

## Calidad de código

El objetivo es producir código que sea fácil de entender y mantener. Escribe el mínimo código necesario para resolver el problema; evita abstracciones especulativas y manejo de errores para casos que no pueden ocurrir.

**Estructura y legibilidad**

- Archivos de 50–300 líneas con un propósito acotado; extrae en módulos cuando un archivo crece o mezcla responsabilidades.
- Valida condiciones de error al inicio de la función y retorna temprano (guard clauses), en lugar de anidar lógica en bloques `if`.
- Evita estructuras de difícil trazabilidad: cadenas largas de llamadas, lambdas complejas, expresiones densas. Descomponlas en variables o funciones intermedias con nombre.
- Extrae helpers con nombre incluso cuando no hay duplicación, si hacen el código más claro.

**Duplicación**

Tolera duplicación hasta dos veces; a la tercera, refactoriza. La duplicación acumulada es deuda que dificulta el mantenimiento.

**Separación de cambios**

Los cambios cosméticos o estructurales van en commits separados de los cambios de comportamiento.

## Documentación

Usa nombres de variables, tipos, funciones y clases que describan el *qué*. Incluye comentarios para lo que no se puede deducir del código: restricciones, decisiones no obvias, trade-offs. Las APIs públicas siempre llevan docstrings.

## Agentes

Los agentes se definen en dos ubicaciones con propósitos distintos:

- `.github/agents/CUSTOM-AGENT-NAME.md` — implementación activa; Copilot los carga como agentes disponibles en el proyecto.
- `material/agents/` — referencia del workshop; documentan el rol y comportamiento esperado de cada agente.

## Prompt files

Los prompt files se guardan en dos ubicaciones con propósitos distintos:

- `.github/prompts/<name>.prompt.md` — implementación activa; Copilot los expone como comandos invocables en el proyecto.
- `material/prompts/` — referencia del workshop; contienen los prompts de ejemplo y documentación de uso.

## Gates de calidad *(proyectos de aplicación)*

- Formato, lint, seguridad, unit e integración corren en cada cambio, siempre.
- Código, tests y auditorías se revisan en contextos separados para evitar sesgos compartidos.
- "Listo" significa que el 100% de los criterios de aceptación están verificados.
