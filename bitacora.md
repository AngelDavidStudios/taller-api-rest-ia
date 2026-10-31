# Bitácora de prompts — Taller API REST con IA Generativa

**Estudiante:** David Rueda · **Asignatura:** ISWZ3104 · **Fecha:** 2026-10-31
**Herramienta de IA:** Claude (Claude Code, modelo Claude Opus 5.5) con MCP de Google Colab, Google Drive y Claude in Chrome.

> Declaración de uso de IA: conforme a la Política de Uso de IA Generativa de la UDLA, el código de este taller
> fue generado con apoyo de Claude y verificado, ejecutado y corregido por el estudiante. El token de GoREST
> nunca se compartió con la IA: se ingresa únicamente con `getpass()` en Colab.

Formato de cada entrada: **Prompt** (qué se pidió) → **Respuesta de la IA** (qué devolvió) → **Ejecución / resultado** → **Corrección** (qué se corrigió y por qué).

---

## Iteración 0 — Planificación

- **Prompt:** se entregó a la IA el PDF del taller y se pidió registrar las instrucciones, armar un plan y definir entregables.
- **Respuesta de la IA:** generó `CLAUDE.md` con el caso, tabla de verificación, entregables, preguntas y rúbrica; propuso un plan en 7 fases (0–6).
- **Corrección:** ninguna; el estudiante definió datos de portada, convención de commits y que la demo la graba él.

## Iteración 1 — Calentamiento en el sandbox (JSONPlaceholder)

- **Prompt:** "Crea el notebook desde cero: una función `mostrar()` que imprima método, URL, código de estado y JSON, y un calentamiento con GET, POST, PUT, PATCH y un GET final contra `jsonplaceholder.typicode.com/posts` para comprobar si el cambio persiste."
- **Respuesta de la IA:** celda 1 (`requests` + `mostrar()`, que maneja respuestas sin cuerpo como 204) y celda 2 (las 5 peticiones con `timeout=10` y una verificación automática de persistencia).
- **Ejecución / resultado:**
  | Método | URL | Código |
  |---|---|---|
  | GET | /posts/1 | 200 OK |
  | POST | /posts | 201 Created (id simulado 101) |
  | PUT | /posts/1 | 200 OK (devuelve solo los campos enviados) |
  | PATCH | /posts/1 | 200 OK (cambia `title`, conserva `body` original) |
  | GET final | /posts/1 | 200 OK — **título original: el cambio NO persistió** |
- **Hallazgo:** JSONPlaceholder responde como si guardara, pero no persiste nada (sandbox). Además, ya se ve la diferencia PUT (reemplaza el recurso completo) vs PATCH (modifica solo el campo enviado).
- **Corrección:** ninguna; funcionó a la primera.

## Iteración 2 — Carga segura del token

- **Prompt:** "Prepara la celda para cargar el token de GoREST con `getpass()`."
- **Respuesta de la IA:** celda que pedía el token con `getpass()`, armaba `HEADERS` con `Authorization: Bearer` e imprimía solo la longitud del token.
- **Revisión del estudiante:** "Los tokens se guardan fuera del código; hay que poner el token en los Secretos de Colab."
- **Corrección:** la celda ahora lee `GOREST_TOKEN` de los Secretos de Colab (`google.colab.userdata.get`) y, si no existe o no tiene acceso, usa `getpass()` como respaldo. Se mantienen ambos porque la rúbrica pide "token vía getpass". El token sigue sin aparecer en el código, las salidas, el log ni el chat con la IA.
