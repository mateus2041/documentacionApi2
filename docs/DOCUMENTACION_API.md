# Documentación de API y funciones críticas

## Información general

- **Servicio:** Puesto Chance API.
- **Protocolo:** HTTP; cuerpos y respuestas JSON.
- **Prefijo de negocio:** `/api/v1`.
- **Autenticación:** JWT en `Authorization: Bearer <token>` para todas las rutas de negocio, excepto `POST /auth/login`. `GET /health` está fuera del prefijo y es público.
- **Especificación:** [openapi.yaml](openapi.yaml), OpenAPI 3.1.0.
- **Swagger UI:** no está instalado ni se publica una ruta interactiva.

La app registra `cors()` y `express.json()`. El middleware de errores devuelve `{ "error": "..." }`; JSON mal formado resulta en 400, ruta inexistente en 404 y error inesperado en 500 con mensaje genérico.

## Endpoints

En las respuestas, `200` indica operación de consulta/actualización completada y `201` creación. Las rutas protegidas pueden devolver `401` si falta o no es válido el JWT. Las restricciones por rol devuelven `403`. Los errores de validación suelen ser `400`, las transiciones incompatibles `409`, los recursos inexistentes `404` y los premios vencidos `410`.

| Método y ruta | Descripción / actor | Parámetros / entrada | Respuesta de éxito | Otros estados relevantes | Autenticación |
|---|---|---|---|---|---|
| `GET /health` | Comprueba que Express responde. Cliente/monitor. | Sin parámetros ni cuerpo. | 200 `{estado:"ok", hora:<ISO>}`. | No se define otra respuesta específica. | Pública |
| `POST /api/v1/auth/login` | Inicia sesión. | JSON requerido: `usuario` (string), `password` (string). | 200 `{token, usuario:{id,usuario,nombre,rol}}`; no devuelve hash. | 400 campos obligatorios; 401 credenciales inválidas; 403 usuario inactivo. | No |
| `GET /api/v1/auth/perfil` | Consulta el perfil actual. | Sin parámetros/cuerpo. | 200 `{id, usuario, nombre, rol}`. | 401 token requerido/inválido o usuario no válido. | Bearer |
| `GET /api/v1/sorteos` | Lista sorteos. | Query opcional `tipo` (CHANCE/BALOTO), `estado` (p. ej. ABIERTO/JUGADO). | 200 arreglo de sorteos. | 401 sin autenticación. | Bearer |
| `GET /api/v1/sorteos/{id}` | Consulta un sorteo. | Path `id`. | 200 objeto sorteo. | 404 no encontrado; 401. | Bearer |
| `POST /api/v1/sorteos` | Crea sorteo; ADMIN. | JSON: `tipo`, `nombre`, `fecha` futura; `acumulado` opcional (Baloto). | 201 sorteo creado en ABIERTO. | 400 datos inválidos; 401/403. | Bearer + ADMIN |
| `POST /api/v1/sorteos/{id}/resultado` | Registra resultado y liquida tiquetes; ADMIN. | Path `id`; JSON `resultado`: string de 4 dígitos para Chance o `{numeros:[5 enteros], superbalota:1..16}` para Baloto. | 200 sorteo JUGADO actualizado. | 400 resultado inválido; 401/403; 404; 409 si ya se jugó. | Bearer + ADMIN |
| `POST /api/v1/tickets/chance` | Vende tiquete de Chance; VENDEDOR. | JSON `sorteoId`, `apuestas` (1–5; cada una `numero`, `valor`, `modalidad` opcional), `cliente` opcional. | 201 ticket con código `CH-...`, total y estado ACTIVO. | 400 validación; 401/403; 404 sorteo; 409 sorteo cerrado/tipo/estado. | Bearer + VENDEDOR |
| `POST /api/v1/tickets/baloto` | Vende tiquete de Baloto; VENDEDOR. | JSON `sorteoId`, `jugadas` (1–10; cada jugada manual `numeros` y `superbalota`, o `aleatoria:true`), `cliente` opcional. | 201 ticket `BL-...`, total y estado ACTIVO. | 400 validación; 401/403; 404 sorteo; 409 sorteo cerrado/tipo/estado. | Bearer + VENDEDOR |
| `GET /api/v1/tickets` | Lista tiquetes por alcance del rol. | Query opcional `fecha` (prefijo de fecha ISO), `estado`. | 200 arreglo. | 401. | Bearer |
| `GET /api/v1/tickets/{codigo}` | Consulta tiquete y resumen de sorteo. | Path `codigo`. | 200 ticket con `sorteo` y `ganador` (booleano tras sorteo, null antes). | 401; 404 si no existe. | Bearer |
| `POST /api/v1/tickets/{codigo}/anular` | Anula ticket; propietario vendedor o ADMIN. | Path `codigo`; JSON con `motivo` requerido. | 200 ticket ANULADO y metadatos de anulación. | 400 motivo requerido; 401/403; 404; 409 estado/sorteo/ventana no válida. | Bearer; autorización adicional en servicio |
| `POST /api/v1/tickets/{codigo}/pagar` | Paga un premio; VENDEDOR. | Path `codigo`; sin cuerpo requerido. | 200 registro `{id,ticketId,codigo,usuarioId,valor,fecha}`; ticket pasa a PAGADO. | 401/403; 404; 409 no jugado/no ganador/estado o pago duplicado; 410 caducado. | Bearer + VENDEDOR |
| `GET /api/v1/caja/resumen` | Resume movimientos del día. | Query opcional `fecha`; `vendedorId` solo es aplicado por el controlador cuando el rol es ADMIN. | 200 resumen con ventas, anulaciones, premios, saldo y `cerrada`. | 401. | Bearer |
| `POST /api/v1/caja/cierre` | Registra cierre del vendedor autenticado. | JSON requerido `efectivoReportado` entero no negativo; `fecha` opcional. | 201 cierre con diferencia y estado. | 400 monto inválido; 401/403; 409 ya cerrado. | Bearer + VENDEDOR |

### Formas de entrada y reglas de dominio

- **Apuesta Chance:** `numero` string de 1–4 dígitos; `modalidad` `DIRECTO` (por defecto) o `COMBINADO`; el combinado requiere 3–4 cifras y no permite todas iguales. `valor` entero entre `APUESTA_MIN_CHANCE` y `APUESTA_MAX_CHANCE`, múltiplo de 100.
- **Jugada Baloto manual:** arreglo `numeros` con exactamente cinco enteros únicos del 1 al 43, y `superbalota` entera del 1 al 16. `{ "aleatoria": true }` genera una jugada.
- El código no aplica un esquema genérico de validación HTTP: los campos extra no se rechazan explícitamente y algunas validaciones existen únicamente en los servicios.
- La consulta `GET /tickets/{codigo}` exige token, pero el servicio no comprueba que el ticket pertenezca al usuario autenticado. La lista sí restringe tickets por vendedor.

## Funciones críticas (JSDoc)

Las funciones siguientes tienen comentarios JSDoc en su archivo de implementación. `AppError` se traduce por el middleware HTTP; las funciones internas no declaran un esquema de retorno estático.

| Función | Descripción y funcionamiento | Parámetros | Retorno | Errores | HU / RF |
|---|---|---|---|---|---|
| `login` — `src/services/authService.js` | Busca usuario, compara contraseña bcrypt y firma JWT con expiración configurable. | `usuario: string`, `password: string`. | `{token, usuario}` con perfil público. | `AppError` 400 por datos ausentes, 401 credenciales inválidas, 403 cuenta inactiva. | HU-001 / RF-001 |
| `validarApuesta` — `src/services/chanceService.js` | Valida número, modalidad, reglas de combinado, rango y múltiplo de apuesta; devuelve objeto normalizado. | `apuesta: Object` (`numero`, `modalidad?`, `valor`). | `{numero, cifras, modalidad, valor}`. | `AppError` (HTTP 400) ante incumplimiento. | HU-003 / RF-007 |
| `registrarResultado` — `src/services/sorteoService.js` | Valida el resultado, cambia el estado del sorteo a JUGADO y calcula premio para cada tiquete activo asociado. | `id: string|number`, `resultado: string|Object`. | Objeto sorteo actualizado. | `AppError` 404 sorteo inexistente; 409 ya jugado; 400 resultado inválido. | HU-002 / RF-006 |
| `venderChance` — `src/services/ticketService.js` | Verifica sorteo/ventana, valida 1–5 apuestas, genera código y guarda ticket activo. | `vendedor: Object`, datos `{sorteoId, apuestas, cliente?}`. | Objeto ticket con total y código. | `AppError` por cantidad/campos; errores heredados de sorteo y apuestas. | HU-003 / RF-007 |
| `venderBaloto` — `src/services/ticketService.js` | Verifica sorteo/ventana, valida 1–10 jugadas, calcula precio y guarda ticket activo. | `vendedor: Object`, datos `{sorteoId, jugadas, cliente?}`. | Objeto ticket con total y código. | `AppError` por cantidad/jugada; errores heredados de sorteo. | HU-004 / RF-008 |
| `pagarPremio` — `src/services/ticketService.js` | Comprueba sorteo jugado, estado ACTIVO, premio y vigencia; agrega pago y cambia ticket a PAGADO. | `codigo: string`, `usuario: Object`. | Registro de pago `{id,ticketId,codigo,usuarioId,valor,fecha}`. | `AppError` 404 no existe; 409 no pagable/duplicado; 410 vencido. | HU-007 / RF-012 |
| `resumen` — `src/services/cajaService.js` | Filtra tickets y pagos del día y calcula ventas, anulaciones, premios, saldo y cierre. | `vendedorId: number`, `fecha?: string` (predeterminada local). | Objeto de resumen. | No declara errores de dominio propios; depende de lectura en memoria. | HU-008 / RF-013 |
| `cerrar` — `src/services/cajaService.js` | Valida efectivo, obtiene resumen, calcula diferencia y agrega registro de cierre. | `vendedorId: number`, `{efectivoReportado: number, fecha?: string}`. | Objeto de cierre con diferencia y estado. | `AppError` 400 monto inválido; 409 cierre duplicado. | HU-008 / RF-014 |

## Semántica general de errores

El middleware devuelve JSON `{error: string}`. Los mensajes de `AppError` con código no especificado usan 400; las excepciones desconocidas se registran en consola y producen 500 con mensaje genérico. El parser de Express convierte JSON mal formado en HTTP 400. La ruta de fallback produce HTTP 404.
