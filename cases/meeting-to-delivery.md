# Meeting to Delivery — de la reunión a la entrega accountable

**Código:** [github.com/rubendariogomeza/meeting-to-delivery](https://github.com/rubendariogomeza/meeting-to-delivery)

**Rol:** producto, diseño del pipeline y desarrollo.
**Stack:** TypeScript, Node, Zod, fflate, Jira REST, MIME.
**Estado:** motor portable completo y probado. Adaptadores de Microsoft 365 documentados, bloqueados por acceso a un tenant.

---

## El problema

Una reunión de una hora produce diez compromisos. Dos días después, nadie sabe cuáles
siguen vivos, cuáles se cerraron y cuáles nunca empezaron. El problema no es la falta de
herramientas: hay Jira, hay actas, hay correos. El problema es que **el estado del trabajo
no es legible**, y lo que no se ve no se gestiona.

Ese fue el problema real en un cliente de healthtech en EE. UU.: un equipo de 21 personas
en cinco frentes, con oficina principal en Miami y personas en Colombia y España, sin
accountability de tareas ni reportes semanales. Se resolvió con JIRA bien administrado y
automatizaciones que hacían visible el avance detenido. Este proyecto convierte esa solución
en un motor que cualquier equipo con Outlook, Teams y Jira puede usar — y que se puede
ejecutar y auditar sin acceso al tenant.

## Qué hace

Dos comandos, un solo contrato de datos:

```
transcripción (.txt / .vtt / .docx)
  └─ meeting  →  informe.md · tareas.json · plan-jira.json · revisión.eml · distribución.eml
tareas.json (+ historial de estados)
  └─ brief    →  brief.md · brief.json
```

El informe tiene cuatro secciones fijas: resumen ejecutivo, acuerdos, decisiones y tareas.
Cada tarea conserva responsable, fecha, prioridad, confianza y **la frase exacta de la que
salió**. El brief semanal responde cuatro preguntas: qué se movió, qué está quieto, qué
necesita una decisión y dónde está concentrado el riesgo.

## La decisión central

**El modelo redacta; el código es dueño de los compromisos.**

Un modelo de lenguaje puede escribir un buen resumen. No se le puede confiar quién prometió
qué ni para cuándo. Por eso el núcleo es determinista:

- Los responsables se resuelven contra la lista de asistentes. Un nombre desconocido se
  convierte en `sin asignar`, nunca en una suposición.
- Las fechas salen de un parser explícito. Si la transcripción no dice nada fechable, la
  tarea queda sin fecha y se marca.
- Los compromisos repetidos en distintas frases se fusionan, y la fusión se registra.
- Cada artefacto se valida contra un esquema antes de que la etapa siguiente lo lea.

El modelo es opcional y solo reescribe las secciones narrativas. Si falla o devuelve algo
inválido, el pipeline cae al informe determinista y deja la advertencia en el registro.

Eso es lo que separa un sistema que se puede operar de una demostración: **límites,
verificación y una salida predecible cuando algo falla.**

## La segunda mitad: el brief

Un tablero muestra todo y no explica nada. El brief elige:

| Sección | Pregunta | Regla |
| --- | --- | --- |
| Se movió | ¿Qué cambió esta semana? | transiciones de estado dentro de la ventana |
| Está quieto | ¿Qué no se mueve? | sin actualización por más de 14 días |
| Necesita decisión | ¿Qué requiere un humano? | sin responsable, vencida o bloqueada más de 7 días |
| Riesgos | ¿Dónde se concentra la entrega? | carga por persona, vencidas, estancadas |

Con los datos sintéticos del demo: 17 tareas abiertas, 8 estancadas, 7 decisiones y 4
riesgos, incluido “Luis Mejía tiene 7 de 17 tareas abiertas (41 %)”. Son cifras de un
fixture, no de un cliente.

## Evidencia

- **41 pruebas** sobre la lógica pura y un recorrido de extremo a extremo: fechas, dueños,
  extracción, fusión de duplicados, esquemas, plan de Jira, correo y brief.
- `npm run demo` produce un informe, nueve tareas, un plan de Jira, dos correos y un brief
  en menos de un segundo, **sin credenciales y sin red**.
- `npm run typecheck` limpio.
- Artefactos de ejemplo publicados en [`examples/`](https://github.com/rubendariogomeza/meeting-to-delivery/tree/main/examples).
- Jira funciona en simulación por defecto; escribe solo con `--apply` y comprueba primero
  si el plan ya existe, para que un reintento no duplique tareas.

## Lo que este proyecto demuestra

No es el código. Es la secuencia que está documentada en el repositorio:

`brief` → `feasibility` → `security-review` → `decisions` → `architecture` → `roster` → `tasks`

La matriz de factibilidad tiene 16 capacidades con estado, riesgo y cómo verificarlas. La
revisión de seguridad compara tres motores de informe por límite de datos, permisos,
retención y auditoría, y termina con las preguntas exactas que hay que hacerle a IT. El
registro de decisiones tiene 24 entradas fechadas, cada una con su consecuencia.

Esa es la misma disciplina de un proyecto de cliente, aplicada a un producto propio. Y es
la parte que un reclutador puede leer en cinco minutos sin abrir el código.

## Límites

- El motor heurístico entiende frases de reunión en español; no es un NLU general.
- La adquisición de la transcripción es manual. La integración con Graph está documentada
  y bloqueada por permisos delegados, consentimiento y licencia de Copilot.
- La detección de duplicados en Jira es por etiqueta de reunión, no por contenido.
- El camino con modelo exige un endpoint compatible con OpenAI.

## Enlaces

- [Repositorio](https://github.com/rubendariogomeza/meeting-to-delivery)
- [Informe de ejemplo](https://github.com/rubendariogomeza/meeting-to-delivery/blob/main/examples/report.md)
- [Brief de ejemplo](https://github.com/rubendariogomeza/meeting-to-delivery/blob/main/examples/brief.md)
- [Contratos de datos](https://github.com/rubendariogomeza/meeting-to-delivery/blob/main/docs/contracts.md)
