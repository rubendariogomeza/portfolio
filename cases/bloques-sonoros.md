# Bloques Sonoros — secuenciador musical tangible

**PWA educativa donde los bloques físicos sobre una pared se convierten en música.**

[Demo en vivo](https://rubendariogomeza.github.io/bloques-sonoros/) ·
[Código](https://github.com/rubendariogomeza/bloques-sonoros)

**Rol:** diseño de producto, arquitectura y dirección de la implementación.
**Stack:** JavaScript nativo (ES modules), Web Audio API, Canvas, Node 20 (solo stdlib).
**Estado:** desplegado y en uso con kits físicos.

---

## El problema

Enseñar música a niños pequeños choca con una barrera concreta: los instrumentos exigen
destreza antes de dar satisfacción. Un niño que no controla sus manos no puede tocar una
melodía, y la mayoría de las apps musicales infantiles sustituyen el instrumento por un
videojuego: el niño toca la pantalla, pero no construye nada.

La idea del proyecto era la contraria: que el gesto físico de **apilar y ordenar** fuera
el que produjera la música, y que el resultado sonara bien sin importar lo que el niño
hiciera. Un aula no es un laboratorio: hay cámaras mal puestas, luz que cambia, niños que
mueven el soporte y profesores sin formación técnica.

## La restricción que definió el diseño

Si cualquier combinación de bloques puede sonar mal, el niño deja de explorar. La decisión
central fue **eliminar el error posible**: toda nota pertenece a una escala pentatónica de
Do o al acorde vigente. No hay forma de desafinar.

Eso convirtió un problema de música en un problema de percepción: detectar qué bloque está
dónde, con suficiente estabilidad para que el sonido no parpadee, y con suficiente
tolerancia para que un aula real funcione.

## Cómo funciona

La escena se lee como una parrilla de 8 filas. Una línea de lectura barre el área de
izquierda a derecha y, al cruzar cada bloque, suena el instrumento de su color en la nota
de su fila. Una pared vacía es silencio total.

**Visión (20 fps).** El frame se reduce a 320 px de ancho y se compara contra un modelo de
fondo por píxel en luma, cromaticidad y saturación. La máscara se umbraliza con histéresis
y se limpia con morfología 3×3. Sobre ella se miden tres cosas según el modo: ocupación y
color por cuadro (Parrilla), ocupación, altura, base, color y simetría (Melodía), o piel en
primer plano YCbCr (Retrato).

**Iluminación.** Un factor global estimado por histograma corrige la exposición. El fondo
guarda reflectancia, no brillo, así que una lámpara que se enciende no rompe la calibración.
Las sombras —mismo matiz, más oscuras— se descartan en lugar de leerse como bloques.

**Recalibración en caliente.** Si la cámara se mueve, se estima la traslación con los
perfiles de luminancia y se desplaza el modelo de fondo. Los bloques siguen sobre la pared
sin detener la música.

**Tiempo.** El transporte es una recta anclada a `AudioContext.currentTime`. El scheduler
programa con 120 ms de lookahead (1,6 s cuando la pestaña pasa a segundo plano). Cambiar
BPM reancla sin saltar.

**Audio.** Cada sample se sintetiza una vez con `OfflineAudioContext` y se reproduce con
`AudioBufferSourceNode`; la melodía se afina con `playbackRate`. Máximo 48 voces.

**Tres modos de juego.** *Parrilla* (por defecto): color → instrumento, fila → grado
pentatónico. *Melodía*: la silueta de la estructura elige registro, densidad, timbre, bajo
y adorno en el compás siguiente. *Retrato*: la cara más grande elige una canción propia y
los gestos la modulan.

## Decisiones de ingeniería

**Cero dependencias en el cliente.** Sin frameworks, sin bundler, sin paso de build. El
navegador carga módulos ES directamente. En un dispositivo de aula que puede ser viejo y
no tener buena conexión, cada capa que se elimina es una falla menos.

**Servidor con la librería estándar de Node.** El único backend es la API de licencias y el
servido de archivos estáticos. Nada que actualizar cada semana.

**Licencia por QR de un solo uso.** El kit físico trae un QR impreso que se canjea una vez
contra el servidor y queda vinculado al dispositivo. El servidor guarda solo
`SHA-256(pepper:token)`, canjea de forma atómica y no permite reutilizar el código. La app
sigue funcionando sin conexión después del canje inicial.

**129 pruebas sobre lógica pura.** Transporte, escalas, mapeos de color y geometría se
prueban sin navegador (`node --test`). Lo que depende de la cámara o del audio se verifica
a mano con una lista de QA.

**Versionado del service worker.** Es cache-first y revalida en segundo plano, así que la
primera carga tras publicar muestra la versión anterior si no se sube `VERSION`. La app
comprueba `sw.js` sin caché y se recarga sola al detectar una versión nueva.

## Límites conocidos

Los documentos del proyecto listan los límites en lugar de esconderlos:

- El desbloqueo vive en el cliente: se puede copiar `localStorage`. La autoridad real es el
  servidor, que impide canjear dos veces el mismo código.
- Detección a 20 fps: entre ~100 y ~300 ms entre mover un bloque y oírlo.
- La calibración exige que la pestaña esté visible (usa `requestAnimationFrame`).
- `BarcodeDetector` no existe en todos los navegadores; hay respaldo con `jsQR` y entrada manual.
- En Retrato solo se sigue la cara más grande, sin estimación de pose: un perfil muy girado
  no suena.
- El registro más grave de la marimba suena apagado; pendiente de ajuste de timbre.

## Por qué este proyecto es relevante para un equipo

No es una demo de visión por computadora. Es un producto con restricciones reales:
dispositivos ajenos, usuarios de seis años, profesores sin soporte técnico y una licencia
que tiene que funcionar sin conexión. Cada decisión anterior —eliminar el error musical,
quitar dependencias, calibrar en caliente, documentar los límites— es una decisión de
producto antes que de código.

Es el mismo criterio que aplico a la entrega de software: **si el sistema no perdona el uso
real, no sirve.**
