# CLAUDE.md — Semana 4 · Taller: Integración mediante API REST con IA Generativa

UDLA · Integración de Sistemas (ISWZ3104) · Actividad 7 · Taller práctico (10%)
Fuente de verdad: `Instrucciones/Taller_Integracion_API_REST_con_IA.pdf` (releerlo ante cualquier duda).

## Forma de trabajo (acordada con el usuario)

- El usuario **supervisa todo**: ante cualquier duda, decisión o ambigüedad, preguntar antes de avanzar.
- Primero se arma un **plan**, luego se ejecuta por fases hasta tener los **entregables**.
- Al final se **revisa todo contra la rúbrica** (sección más abajo) antes de entregar.
- Herramientas:
  - **Colab MCP** (`mcp__colab-mcp__*`) para desarrollar y ejecutar el notebook.
  - **Google Drive MCP** (`mcp__claude_ai_Google_Drive__*`) para el notebook, `integracion.log` y archivos compartidos.
  - **Claude in Chrome** (`mcp__claude-in-chrome__*`) para que Claude tome las **capturas** del informe (cada operación con su código de estado, incluido el camino de error).
  - **Claude Docs** para el **informe final** (formato UDLA).
- Regla de oro del taller: **un cambio a la vez** — generar, correr, probar, siguiente.

## Datos y decisiones del proyecto

- **Estudiante:** David Rueda · **Asignatura/paralelo:** ISWZ3104 · **Entrega:** 2026-10-31.
- **Notebook:** se crea desde cero (no se usa `taller-apis-rest-estudiante.ipynb`).
- **Demo (4 pts):** la graba el usuario; Claude solo toma las capturas del informe.
- **Token GoREST:** el usuario ya lo tiene (lo ingresa él con `getpass`).
- **Repositorio GitHub:** sí, **público** (cuenta `AngelDavidStudios`), con `.ipynb`, `integracion.log` y README.
  **El PDF de `Instrucciones/` NO se sube** (está en `.gitignore`).
- **Validación previa (firma personal de David Rueda)** para POST/PUT, además de email con formato válido,
  `gender ∈ {male, female}` y `status ∈ {active, inactive}`:
  1. **Nombre completo:** ≥ 2 palabras, solo letras y espacios, cada palabra con mayúscula inicial, 5–60 caracteres.
  2. **Dominio corporativo ficticio:** el email debe terminar en `@techdistribuidora.com` (regla de negocio; GoREST no verifica dominios).
  3. **Sin emails desechables:** rechazar mailinator.com, tempmail.com, yopmail.com, etc. (se valida antes que el dominio, con mensaje propio).
  - Para que el email sea único en GoREST: `nombre.apellido.<timestamp>@techdistribuidora.com`.

## Convención de commits (obligatoria)

- Commit **al cerrar cada proceso/fase**.
- **Una sola línea**, **en inglés**, empezando con **✨**. Ej.: `✨ Add JSONPlaceholder warm-up notebook cells`
- **Sin** `Co-Authored-By` ni ninguna otra firma/atribución (esta regla del usuario prevalece).

## Reglas de seguridad / honestidad académica (obligatorias)

- **El token de GoREST NUNCA** se escribe en el código, en una celda, en el chat, en el log ni en el informe.
  Se guarda en **Secretos de Colab** con el nombre `GOREST_TOKEN` (`google.colab.userdata.get`); si no existe,
  respaldo con `getpass()` (la rúbrica exige "token vía getpass", por eso se mantienen ambos).
  **Lo configura el usuario.** Claude no debe pedirlo, leerlo ni teclearlo.
- Antes de compartir el notebook: verificar que ninguna celda ni salida contenga el token.
- El uso de IA es obligatorio declararlo. La **bitácora de prompts** es evidencia académica: registrar cada
  iteración (qué se pidió, qué devolvió la IA, qué se corrigió y por qué) en `bitacora.md`.
- El usuario debe poder **explicar cualquier bloque del código** en la defensa oral → código simple y comentado.

## Caso: TechDistribuidora + CRM (GoREST)

Patrón: **Remote Procedure Invocation** vía API REST (`requests` en Python, Google Colab).

1. **Calentamiento sandbox** — JSONPlaceholder (`https://jsonplaceholder.typicode.com/posts`): GET, POST, PUT, PATCH y un GET final para comprobar que **no persiste**.
2. **Integración real** — GoREST (`https://gorest.co.in/public/v2`), token Bearer, datos **sí persisten**.

| Operación          | Método y ruta                          | Código esperado |
|--------------------|----------------------------------------|-----------------|
| Consultar clientes | GET /users?page=1&per_page=5           | 200 OK          |
| Registrar cliente  | POST /users (guardar id)               | 201 Created     |
| Verificar registro | GET /users/{id}                        | 200 OK          |
| Reemplazar datos   | PUT /users/{id}                        | 200 OK          |
| Cambiar estado     | PATCH /users/{id} (status → inactive)  | 200 OK          |
| Dar de baja        | DELETE /users/{id}                     | 204 No Content  |
| Comprobar baja     | GET /users/{id}                        | 404 Not Found   |
| Camino de error    | POST sin token / con datos incompletos | 401 / 422       |

En cada paso mostrar: método, URL, código de estado y JSON.

### Ampliación obligatoria (segundo prompt / iteración)

1. Log en `integracion.log`: fecha, hora, método, URL y código de estado de **cada** petición.
2. Validación previa a POST/PUT: formato de email, `gender` y `status` permitidos + **regla personal** (firma del estudiante; a definir con el usuario y documentar).
3. Manejo de errores **401, 404, 422, 429** con mensaje claro en pantalla y en log, sin detener el programa.
4. Recurso anidado: `POST /users/{id}/todos` y marcarla completada con `PATCH /todos/{id_tarea}`.
5. Probar con **token inválido** y con **datos incompletos**.

## Entregables

1. **Informe formato UDLA** (Claude Docs) con:
   - Paso a paso para desplegar la solución.
   - Evidencias: capturas de cada operación con su código de estado, incluido el camino de error.
   - Bitácora de prompts (qué se pidió, qué devolvió la IA, qué se corrigió en cada iteración).
   - Respuestas a las 5 preguntas (abajo).
   - Conclusiones y recomendaciones.
2. **Enlace al notebook de Colab** compartido con permiso de lectura, sin token (opcional: repo GitHub con `.ipynb` + README).
3. **Archivo `integracion.log`** generado por la integración.

### Preguntas a responder en el informe

1. ¿Qué patrón de integración se usó y en qué se diferencia del patrón File Transfer (actividad anterior)?
2. ¿Qué diferencia hubo entre JSONPlaceholder y GoREST y qué implica para probar una integración?
3. ¿Diferencia entre PUT y PATCH? (con evidencia de la ejecución)
4. Mejoras para producción (reintentos, manejo de credenciales, límites de peticiones).
5. ¿Qué errores cometió la IA y cómo se detectaron y corrigieron?

## Rúbrica (10 pts) — checklist de revisión final

- [ ] **4 pts** Funcionamiento base: GET, POST, PUT, PATCH, DELETE contra GoREST con códigos esperados y token vía `getpass` (demo en vivo o video corto).
- [ ] **2 pts** Ampliación: log, validación previa, manejo de errores, recurso anidado de tareas (probado con token inválido y datos incompletos).
- [ ] **2 pts** Informe formato UDLA con bitácora de prompts completa.
- [ ] **1 pt** Defensa oral: el estudiante explica cualquier bloque del código.
- [ ] **1 pt** Notebook compartido (y repo opcional) reproducible y sin credenciales expuestas.

## Estructura de archivos del proyecto

- `Instrucciones/` — PDF del taller.
- `CLAUDE.md` — este archivo.
- `bitacora.md` — bitácora de prompts e iteraciones (se crea en la fase de desarrollo).
- `evidencias/` — capturas tomadas con Claude in Chrome (nombres descriptivos: `01_get_users_200.png`, …).
- `entregables/` — copia local del `.ipynb`, `integracion.log` y enlaces finales.
