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

## Iteración 4 — Prompt de iteración v2: ampliación obligatoria

- **Prompt v2:**
  ```
  Modifica el programa:
  1. Registra cada petición en integracion.log con fecha, hora, método, URL y código de estado.
  2. Antes de enviar un POST o PUT, valida que el email tenga formato correcto y que gender y status
     tengan valores permitidos, más mis reglas: nombre completo (nombre y apellido, solo letras,
     mayúscula inicial, 5-60 caracteres), email corporativo @techdistribuidora.com y sin emails desechables.
  3. Maneja los errores 401, 404, 422 y 429 con un mensaje claro en pantalla y en el log, sin detener el programa.
  4. Crea una tarea para el usuario con POST /users/{id}/todos y márcala como completada con PATCH /todos/{id_tarea}.
  Un cambio a la vez: generar, correr, probar, siguiente.
  ```
- **Respuesta de la IA:** una celda por cambio, cada una con su prueba al final:
  - Celda 10, `registrar()`: logger con `FileHandler` (sin duplicar handlers al volver a ejecutar). Nunca registra headers, así el token no llega al log.
  - Celda 11, `validar_cliente()`: devuelve la lista de errores. Se probó con 6 casos (válido, incompleto, nombre inválido, email desechable, no corporativo, valores no permitidos) y todos dieron lo esperado.
  - Celda 12, `peticion()` + `manejar_respuesta()`: función central que valida, envía, maneja errores (incluidos los de red) y registra. Para el 422 muestra los campos que rechazó GoREST y para el 429 el tiempo de espera.
  - Celda 13: flujo completo v2 con la tarea anidada.
  - Celda 14: pruebas del camino de error.
  - Celda 15: muestra y descarga `integracion.log`.
- **Errores de la IA detectados y corregidos:**
  1. **Hora del log en UTC.** La primera línea salió con `13:15:07` cuando en Ecuador eran las 08:15, porque Colab corre en UTC. *Detección:* se comparó con la hora local. *Corrección:* `os.environ["TZ"] = "America/Guayaquil"; time.tzset()` y se eliminó la línea con hora UTC del log.
  2. **Celda temporal fallida.** Al reiniciar el log, la IA ejecutó tres acciones a la vez sin esperar el resultado. La primera falló (`NameError: os`), así que el archivo no se borró y la línea UTC siguió ahí. *Corrección:* otra celda temporal filtró esa línea. Lección: aplicar también aquí "un cambio a la vez".
  3. **Zona horaria de `due_on` en la tarea.** La IA envió `2026-11-07T08:16:38` sin offset y GoREST lo guardó como `08:16:38+05:30` (hora de la India): vencimiento corrido 10,5 h. *Detección:* revisando el JSON de respuesta. *Corrección:* `.astimezone().isoformat()` envía `-05:00` y GoREST lo guardó como `18:47:02+05:30`, que equivale a 08:17:02 en Ecuador.
  4. **El 429 no se provoca contra la API real** (sería saturar un servicio público). Se demuestra su manejo con una respuesta simulada que queda marcada como "SIMULADA" en pantalla y en el log.
- **Resultado del flujo v2 (cliente 8694150, tarea 112087):** GET lista 200 · POST 201 · GET 200 · PUT 200 · PATCH 200 · POST todos 201 · PATCH todo 200 (pending → completed) · GET todos 200 · DELETE 204 · GET 404 (manejado).
- **Camino de error:**
  | Prueba | Resultado |
  |---|---|
  | POST sin token | **401** "Authentication failed" |
  | POST con token inválido | **401** "Invalid token" |
  | POST incompleto con validación | **No enviado**: faltan email, gender, status |
  | POST incompleto sin validación | **422**: email/gender/status "can't be blank" |
  | POST con email no corporativo | **No enviado**: firma personal |
  | GET a cliente eliminado | **404** manejado |
  | 429 simulado | Mensaje claro + "reintentar en 2 s" |
- **Log:** `entregables/integracion.log` (28 líneas, sin token).
