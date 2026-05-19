# Template: Caso de Uso Técnico

## Encabezado
- **ID:** TUC-[número]
- **Nombre:** [nombre descriptivo — componente + acción, ej. "Autenticar usuario vía OAuth 2.0"]
- **UC de negocio asociado:** UC-[número] *(si aplica)*
- **Componente responsable:** [servicio o módulo que implementa este caso de uso]
- **Fecha:** [YYYY-MM-DD]
- **Estado:** `borrador` <!-- borrador | aprobado | en proceso | parcial | completo -->

## Contexto técnico

[Descripción de la interacción a implementar: qué sistema o componente inicia, qué otros participan, y qué problema técnico resuelve. Sin lógica de negocio — solo lo necesario para entender el flujo.]

## Precondiciones técnicas

- [ej. token JWT válido presente en el header `Authorization`]
- [ej. servicio de caché disponible y con conexión activa]

## Postcondiciones técnicas

- [estado del sistema, base de datos o caché al finalizar exitosamente]

## Contrato de la interfaz

### Request
```
[Método HTTP / evento / mensaje]
[Endpoint / topic / cola]

Headers:
  [header]: [tipo y descripción]

Path params:
  [param]: [tipo]

Query params:
  [param]: [tipo, requerido/opcional]

Body (JSON / schema):
{
  "[campo]": "[tipo]"  // descripción
}
```

### Response exitosa
```
HTTP [código]

{
  "[campo]": "[tipo]"  // descripción
}
```

### Responses de error
| Código | Condición | Body / mensaje |
|--------|-----------|----------------|
|        |           |                |

## Diagrama de secuencia

```
[Actor/Client] -> [Componente] -> [Dependencia]
     |                 |               |
     |-- request ----> |               |
     |                 |-- llamada --> |
     |                 |<-- resp ----- |
     |<-- response --- |               |
```

## Flujo principal

| Paso | Componente | Acción técnica |
|------|------------|----------------|
| 1    |            |                |

## Flujos alternativos

| ID | Condición | Descripción técnica |
|----|-----------|---------------------|
| A1 |           |                     |

## Manejo de errores y excepciones

| ID | Condición | Comportamiento | Código de error |
|----|-----------|----------------|-----------------|
| E1 |           |                |                 |

## Requerimientos no funcionales

| Atributo | Valor objetivo | Notas |
|----------|----------------|-------|
| Latencia (p99) |          |       |
| Throughput     |          |       |
| Disponibilidad |          |       |
| Timeout        |          |       |

## Seguridad

- **Autenticación:** [mecanismo requerido]
- **Autorización:** [roles o permisos necesarios]
- **Datos sensibles:** [cómo se tratan — cifrado, masking, auditoría]
- **Rate limiting:** [aplica? límites]

## Dependencias técnicas

| Componente / Servicio | Tipo | Versión mínima | Notas |
|-----------------------|------|----------------|-------|
|                       | Interno / Externo |    |       |

## ADRs relacionados

| ADR | Decisión relevante |
|-----|--------------------|
|     |                    |

## Criterios de aceptación técnicos

| ID    | Criterio | Cómo verificar | Estado |
|-------|----------|----------------|--------|
| TAC-1 |          | Test / contrato / métrica | borrador |

## Control de versiones
| Versión | Fecha | Autor | Cambios |
|---------|-------|-------|---------|
| 0.1     |       |       | Borrador inicial |
