# Sound of Self — Cloudflare release v59

## Estado de publicación — 15 de octubre de 2026

Desplegada directamente mediante la API de Cloudflare en https://sound-of-self.darioariza21.workers.dev/play.html. El Worker tiene sus recursos y la base D1 nueva tiene sus tablas y siete migraciones registradas. TEACHER_ACCESS_KEY quedó configurada con autorización del propietario. La clave se entregó en un archivo privado y no está en este repositorio ni en este ZIP. El inicio de sesión real en un dispositivo sigue pendiente de prueba. La verificación HTTP desde el entorno automatizado recibe Cloudflare 403/1010, por lo que todavía no se comprobó la carga real, el audio, la cámara o el guardado externo desde un dispositivo. Prueba en un iPad del colegio antes de usarlo con alumnos.

Se conserva la publicación original en ChatGPT Sites. Las actualizaciones externas se publican por petición; no existe sincronización automática con Sites ni con GitHub. El repositorio GitHub contiene documentación, no una copia completa del código todavía.


Release: Sites v59, 15 October 2026. Source commit: 5ecd9a9b218275d863445ebd60273c8d2b56d67b.

## Qué incluye

Todos los archivos públicos de la versión recuperada: juegos, imágenes, música, sonidos, bibliotecas 3D y modelo de seguimiento de manos, además del servidor/API, esquema y siete migraciones SQL. Incluye un adaptador para ejecutar la aplicación fuera de Sites.

Este paquete es el código de un repositorio para **Cloudflare Workers + D1**. No se sube entero a “Upload your static files”; ese método solo serviría los archivos y no configuraría la base de datos ni las API.

## Configuración inicial

1. Sube el contenido de esta carpeta a tu repositorio de GitHub (sin node_modules, .env, dist ni .wrangler).
2. Crea una base de datos Cloudflare D1 llamada `sound-of-self`. En una computadora con Node >=22.13, instala las dependencias con `npm ci` y autentica Wrangler con `npx wrangler login`.
3. Ejecuta `npx wrangler d1 create sound-of-self` si aún no creaste la base desde el panel. Copia el `database_id` real en `wrangler.jsonc`, sustituyendo el UUID de ejemplo. Puedes usar `node scripts/configure-cloudflare.mjs --database-id UUID`.
4. Inicializa las tablas con `npm run db:migrate`. Solo sobre la nueva base de datos externa. No apuntes a otra base existente sin revisar primero sus migraciones.
5. Confirma el cambio de `wrangler.jsonc` en GitHub. En Cloudflare: **Workers & Pages → Create application → Connect GitHub**, selecciona el repositorio y la rama de producción `main`. Elige el flujo de Worker, no una exportación HTML de Pages.
6. Comando de build: `npm run build:cloudflare`. Comando de despliegue: `npx wrangler deploy --config dist/server/wrangler.json`. La instalación de dependencias usa el package-lock incluido. La raíz del proyecto es `/`.
7. En el Worker agrega el secreto `TEACHER_ACCESS_KEY`: una clave aleatoria de al menos 32 caracteres. Puedes crearla localmente con `node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"` y guardarla en el gestor de contraseñas. No la pongas en GitHub ni en el navegador de los alumnos. El comando alternativo es `npx wrangler secret put TEACHER_ACCESS_KEY --config dist/server/wrangler.json` después del primer despliegue.
8. Abre `/teacher-login` e ingresa esa clave para administrar tus clases. Los alumnos conservan el acceso por código de clase + avatar. Esta configuración externa tiene un único propietario docente; no es un sistema de registro para múltiples docentes independientes.
9. Prueba el enlace `https://<worker>.<cuenta>.workers.dev/play.html` en un iPad de estudiantes. Que cloudflare.com abra no confirma que el filtro permita el subdominio del sitio. Usa la revisión de Helpdesk si lo bloquea.

La configuración del enlace GitHub–Cloudflare y los secretos requieren acceso a tu cuenta de Cloudflare. El paquete fue desplegado directamente; la integración continua de GitHub sigue pendiente.

## Flujo de actualización

Experimenta en la versión de ChatGPT Sites. Cuando elijas publicar una versión, lleva esos cambios al repositorio externo. Cada push a la rama de producción conectada inicia el build y despliegue de Cloudflare. Los cambios de Sites y la automatización semanal de mensajes de Sites **no se copian automáticamente** a este repositorio: hay que sincronizar la versión elegida. Mantén el adaptador `deployment/` y `wrangler.jsonc` al traer cambios de Sites.

## Base de datos e identidad

- La base nueva empieza vacía. Las clases, resultados y sesiones que ya están guardados en Sites no están en este ZIP y no se trasladan solos.
- Se conservan las siete migraciones originales y las API; no se finge guardado con respuestas locales.
- Fuera de Sites, el inicio de sesión de ChatGPT no lo proporciona automáticamente Cloudflare. El adaptador usa una sesión firmada del propietario docente y elimina las cabeceras de identidad enviadas por el navegador. La clave nunca se envía a los alumnos ni se incluye en los archivos públicos.
- La ruta anterior `/signin-with-chatgpt` se mantiene como alias local de la pantalla docente, para conservar compatibilidad con los enlaces existentes. No intenta abrir ChatGPT.
- La clave ya está configurada en esta instalación; futuros despliegues deben conservar ese secreto.
- El modo Live se conserva como demo, igual que en el código fuente; no se convierte en una nueva función de producción. `TEACHER_STUDIO_PREVIEW=0` conserva el estudio experimental oculto.

## Verificación y límites

Se verifican build de producción, TypeScript, autenticación externa y migraciones SQLite. El manifiesto `FILE-MANIFEST.json` contiene tamaño y SHA-256 de todos los archivos incluidos. Los archivos no incluyen credenciales de GitHub, Cloudflare ni Sites. Las dependencias se instalan desde package-lock; no se empaqueta node_modules.

Falta la prueba en Cloudflare con D1 real y un iPad físico: no se puede afirmar que la cámara, el filtro escolar y la persistencia externa estén confirmados hasta realizarla. La migración del código no equivale a migrar los registros actuales de Sites.

## Referencias oficiales

- https://developers.cloudflare.com/workers/ci-cd/builds/
- https://developers.cloudflare.com/workers/vite-plugin/
- https://developers.cloudflare.com/d1/reference/migrations/
- https://developers.cloudflare.com/workers/configuration/secrets/
