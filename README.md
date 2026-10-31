# Taller: Integración mediante API REST con IA Generativa

**UDLA · Integración de Sistemas (ISWZ3104) · Actividad 7** · Estudiante: **David Rueda** · Octubre 2026

Integración del caso *TechDistribuidora* con su CRM (**GoREST**) usando el patrón **Remote Procedure Invocation** sobre API REST, desarrollada en Google Colab con apoyo de IA generativa (Claude).

- GET, POST, PUT, PATCH y DELETE autenticados con token **Bearer**, verificando cada código de estado.
- Calentamiento previo en el sandbox **JSONPlaceholder** (no persiste los cambios).
- Ampliación: log `integracion.log`, validación previa con reglas propias, manejo de errores **401 / 404 / 422 / 429** y recurso anidado de tareas (`/users/{id}/todos`).

## Contenido del repositorio

| Ruta | Qué contiene |
| --- | --- |
| [`notebook/Taller_API_REST_DavidRueda.ipynb`](notebook/Taller_API_REST_DavidRueda.ipynb) | Notebook completo con las salidas de la ejecución |
| [`entregables/integracion.log`](entregables/integracion.log) | Log generado por la integración (28 peticiones) |
| [`evidencias/`](evidencias/) | Capturas de cada operación con su código de estado, incluido el camino de error |
| [`bitacora.md`](bitacora.md) | Bitácora de prompts: qué se pidió, qué devolvió la IA y qué se corrigió |
| [`CLAUDE.md`](CLAUDE.md) | Instrucciones y decisiones del proyecto usadas por el asistente de IA |

Notebook en Colab (solo lectura): https://colab.research.google.com/drive/1AhVf7ZG6cOUF6s2lRIoFM21y8uBUl8Ua

## Cómo ejecutarlo

1. Obtén tu **Access Token** en [gorest.co.in/consumer/login](https://gorest.co.in/consumer/login).
2. Abre el notebook en Google Colab (enlace de arriba, o `Archivo → Subir cuaderno` con el `.ipynb`).
3. En el panel 🔑 **Secretos** crea `GOREST_TOKEN` con tu token y activa **Acceso al notebook**.
   Si no lo haces, la celda 3 lo pedirá con `getpass()`. **Nunca escribas el token en una celda.**
4. `Entorno de ejecución → Ejecutar todo`. Cada ejecución crea un cliente de prueba con email único y lo elimina al final.
5. La celda 15 muestra y descarga `integracion.log`.

## Resultados

| Operación | Método y ruta | Esperado | Obtenido |
| --- | --- | --- | --- |
| Consultar clientes | `GET /users?page=1&per_page=5` | 200 | ✅ 200 |
| Registrar cliente | `POST /users` | 201 | ✅ 201 |
| Verificar registro | `GET /users/{id}` | 200 | ✅ 200 |
| Reemplazar datos | `PUT /users/{id}` | 200 | ✅ 200 |
| Cambiar estado | `PATCH /users/{id}` | 200 | ✅ 200 |
| Dar de baja | `DELETE /users/{id}` | 204 | ✅ 204 |
| Comprobar baja | `GET /users/{id}` | 404 | ✅ 404 |
| Camino de error | `POST` sin token / datos incompletos | 401 / 422 | ✅ 401 / 422 |
| Tarea anidada | `POST /users/{id}/todos` · `PATCH /todos/{id}` | 201 / 200 | ✅ 201 / 200 |

## Seguridad

- El token vive en los **Secretos de Colab** y no aparece en el código, en las salidas, en el log ni en este repositorio.
- El log nunca registra headers.
- El error 429 se demuestra con una respuesta **simulada** para no saturar la API pública.

## Uso de IA

Conforme a la Política de Uso de IA Generativa de la UDLA: el código se generó con apoyo de **Claude** (Anthropic) mediante Claude Code y el MCP de Google Colab. El estudiante especificó, verificó y corrigió cada iteración; el detalle está en [`bitacora.md`](bitacora.md). El token nunca se compartió con la IA.
