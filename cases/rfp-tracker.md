# Monitor de contratación pública — agente de automatización

**Vigilancia automática de un portal de compras gubernamental, con detección de cambios,
archivado de documentos y alertas por correo.**

**Rol:** diseño, desarrollo y operación.
**Stack:** PowerShell, Windows Task Scheduler, SMTP, estado en JSON.
**Código:** privado (construido en el contexto de un cliente). Este documento describe el
problema y las decisiones, no la implementación.

---

## El problema

Las oportunidades de contratación pública aparecen, cambian y se amplían sin aviso. Un
anexo nuevo puede alterar el alcance de una propuesta; una respuesta a preguntas puede
invalidar un supuesto de costos. La única forma de enterarse era que alguien entrara al
portal a revisar, y eso compite con todo lo demás que esa persona tiene que hacer.

El costo de no enterarse no es una molestia: es una propuesta enviada con información
vencida, o una oportunidad que se cerró sin haber sido evaluada.

## Lo que construí

Un agente que corre solo, cada hora, y responde una pregunta simple: **¿qué cambió desde
la última vez?**

1. **Extracción.** Lee la sección de RFP activas del portal y normaliza cada ítem
   (solicitud, documento, fecha, enlace).
2. **Comparación.** Un motor de diferencias contrasta el estado actual contra la fotografía
   persistente y clasifica: RFP nueva, documento nuevo, documento modificado.
3. **Archivado.** Descarga y organiza los adjuntos (PDF, Excel, Word) en carpetas por
   solicitud, resolviendo los enlaces indirectos del visor de Office.
4. **Alerta.** Si hay novedad, envía un correo con enlaces directos y estado. Si no hay
   nada nuevo, manda un resumen cada tres horas en lugar de un correo por ejecución.
5. **Registro.** Todo queda en un histórico con fecha, para poder reconstruir qué se sabía
   y cuándo.

## Decisiones que importan

**Estado persistente, no re-lectura.** El sistema no "reporta lo que ve": reporta la
*diferencia* contra lo que ya sabía. Sin estado, cada ejecución sería una lista completa y
la alerta perdería sentido.

**Filtro estricto de novedades.** La regla es avisar solo de lo que es nuevo, no de lo que
sigue existiendo. Un sistema que notifica de más se ignora en una semana; uno que notifica
poco se lee.

**Resumen periódico en vez de silencio.** Cuando no hay cambios, el silencio total genera
duda ("¿estará corriendo?"). Un resumen cada tres horas confirma que el vigía sigue vivo sin
convertirse en ruido.

**Credenciales fuera del repositorio.** La contraseña de aplicación SMTP se lee de una
variable de entorno. El archivo de configuración versionado solo tiene valores vacíos.

**Operación sin consola.** Instalación, desinstalación y diagnóstico son scripts separados,
más una tarea programada de Windows. La persona que lo usa no necesita abrir una terminal.

## Límites conocidos

- Depende de la estructura HTML del portal: un rediseño del sitio rompe el extractor y hay
  que ajustarlo. Es el costo de no tener una API.
- El correo es el canal de salida, no hay integración con Slack ni con un tablero.
- Todo corre en Windows. La lógica es portable, el programador de tareas no.

## Por qué es un buen caso

Demuestra tres cosas que no se ven en un CV: que sé leer un proceso manual y encontrar
dónde se rompe, que puedo construir la automatización completa —extracción, estado,
diferencia, notificación— y que la opero en producción, con registro y diagnóstico, en
lugar de entregar un script y desaparecer.

Es exactamente el patrón que vendo como servicio: **vigilar lo que nadie tiene tiempo de
vigilar, y avisar solo cuando importa.**
