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

## Iteración 3 — Prompt maestro v1: integración con GoREST

- **Prompt maestro v1:**
  ```
  Actúa como desarrollador de integraciones.
  Construye un programa en Python para Google Colab que consuma la API REST
  de GoREST (https://gorest.co.in/public/v2) usando la librería requests:
  1. Use el token (Secretos de Colab / getpass) en el header Authorization: Bearer.
  2. Liste 5 usuarios con GET usando los parámetros page y per_page.
  3. Cree un usuario con POST (name, email único, gender, status) y guarde su id.
  4. Consulte ese usuario con GET por id.
  5. Reemplace todos sus datos con PUT.
  6. Cambie solo su status a inactive con PATCH.
  7. Lo elimine con DELETE y compruebe que un GET posterior devuelve 404.
  En cada paso muestra el método, la URL, el código de estado y el JSON.
  Entrégame el código en celdas separadas y cómo ejecutarlo.
  ```
- **Respuesta de la IA:** una celda por operación (celdas 4 a 9), reutilizando `mostrar()`. Usa `params=` para la paginación y muestra los headers `x-pagination-*`, genera un email único con `time.time()` y el dominio `@techdistribuidora.com`, guarda `USER_ID` solo si el POST devuelve 201, y usa `timeout=15` en todas las peticiones.
- **Primer intento:** la celda 4 falló con `NameError: name 'BASE_URL' is not defined`.
  - **Diagnóstico:** la IA revisó el notebook y vio que la celda 3 (token) tenía `execution_count: null`. El estudiante la había ejecutado en otra pestaña o copia del notebook, que tiene otro entorno de ejecución.
  - **Corrección:** el estudiante activó *Acceso al notebook* para `GOREST_TOKEN` en este notebook y ejecutó las celdas 1 y 3. No hubo que cambiar el código. Lección: las variables viven en el entorno de ejecución, así que las celdas se ejecutan en orden y en el mismo notebook.
- **Resultado (cliente id 8694141):**
  | Operación | Método y ruta | Esperado | Obtenido |
  |---|---|---|---|
  | Consultar clientes | GET /users?page=1&per_page=5 | 200 | **200 OK** (5 usuarios, 588 páginas) |
  | Registrar cliente | POST /users | 201 | **201 Created** (id 8694141) |
  | Verificar registro | GET /users/8694141 | 200 | **200 OK** |
  | Reemplazar datos | PUT /users/8694141 | 200 | **200 OK** (cambian name y email) |
  | Cambiar estado | PATCH /users/8694141 | 200 | **200 OK** (solo status → inactive) |
  | Dar de baja | DELETE /users/8694141 | 204 | **204 No Content** |
  | Comprobar baja | GET /users/8694141 | 404 | **404 Not Found** ("Resource not found") |
- **Evidencia PUT vs PATCH:** el PUT envió los 4 campos y reemplazó name ("David Rueda" → "David Alejandro Rueda") y email. El PATCH envió solo `{"status": "inactive"}`, y name, email y gender quedaron como los dejó el PUT.
