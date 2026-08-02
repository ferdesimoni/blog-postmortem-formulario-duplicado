Post-mortem: El formulario de contacto enviaba emails duplicados
[Fernando Desimoni · [2/08/2026]
Contexto
Este sitio es una web personal/de proyecto donde los visitantes pueden escribir a través de un formulario de contacto simple (nombre, email, mensaje). El envío dispara dos acciones: guardar el mensaje en la base de datos y enviar una notificación por email tanto al visitante (confirmación) como a mí (aviso de nuevo contacto).
Problema
Varios usuarios reportaron (y yo mismo lo confirmé al probar el formulario) que al enviar un solo mensaje, recibían entre 2 y 4 emails de confirmación idénticos. Del lado del administrador, esto generaba spam en mi bandeja y hacía parecer que el sitio tenía un comportamiento poco confiable.
Acciones (análisis post-mortem)
Línea de tiempo
Día 0, 10:15 — Un usuario reporta por redes sociales que recibió 3 correos de confirmación tras un solo envío.
Día 0, 10:40 — Reproduzco el bug enviando el formulario yo mismo: confirmo 2-3 envíos duplicados según la velocidad de conexión.
Día 0, 11:00 — Reviso el código del handler del formulario y los logs del servicio de email.
Día 0, 11:30 — Identifico la causa raíz (ver abajo).
Día 0, 12:15 — Aplico el fix, lo pruebo en local y en staging.
Día 0, 13:00 — Deploy a producción y verificación final.
Causa raíz
El botón de "Enviar" no deshabilitaba su estado mientras la petición estaba en curso. Si el usuario hacía doble clic (algo muy común) o la conexión era lenta y el usuario reintentaba pensando que no había funcionado, el formulario disparaba múltiples requests al mismo endpoint, y cada request generaba su propio email. No había ninguna validación de idempotencia del lado del servidor tampoco.
En resumen: falta de protección tanto en el frontend (doble clic) como en el backend (sin control de duplicados).
Impacto
Molestia para los usuarios que recibían correos redundantes.
Percepción de baja calidad/confiabilidad del sitio.
Sin impacto en datos: los mensajes no se duplicaban en la base, solo los emails.
Acciones correctivas y preventivas
Frontend: deshabilitar el botón de envío apenas se hace clic, hasta recibir respuesta del servidor (o mostrar un spinner de carga).
Backend: agregar una validación de idempotencia — si llega un request idéntico (mismo email + mismo mensaje) dentro de una ventana de 30 segundos, se ignora el duplicado.
Testing: agregué un test automático que simula doble envío rápido y verifica que solo se dispare un email.
Aprendizajes
Un bug de UX simple (falta de feedback visual en un botón) puede convertirse en un problema de backend si no hay protección en ambas capas.
La idempotencia no es solo un concepto de APIs "serias" — cualquier formulario público debería contemplarla.
Reproducir el bug antes de arreglarlo evitó que aplicara un fix a ciegas que no resolviera la causa real.
Evidencia de control de versiones
Issue: [Post-Mortem] Formulario de contacto envía emails duplicados — [enlace al issue]
Pull Request con la solución y esta entrada de blog: [enlace al PR]
Commits relevantes: [enlace a commits]
Reflexión sobre feedback radicalmente sincero
Al documentar este incidente, evité el impulso de suavizar la causa raíz (mi propio código no tenía protección básica de doble envío) solo para "quedar mejor" en el post. Aplicar feedback radicalmente sincero conmigo mismo significó ser directo sobre el error —sin protección ni en frontend ni en backend, algo evitable desde el diseño— pero también constructivo: en vez de quedarme en la autocrítica, prioricé documentar la causa con evidencia y transformar el incidente en una mejora concreta (idempotencia + test automatizado) que previene que se repita. Ese equilibrio entre honestidad y orientación a la mejora es, justamente, lo que hace útil a un post-mortem.
