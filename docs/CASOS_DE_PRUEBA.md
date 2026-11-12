# Casos de prueba y estado QA

Los casos CP-001 a CP-020 corresponden a pruebas Jest encontradas en el repositorio y ejecutadas el 12-11-2026: **20 aprobados, 0 fallidos**, en 5 suites. Los casos CP-021 en adelante son propuestas para cobertura faltante: **pendientes de ejecución**. Los pasos propuestos son manuales/automatizables y no representan resultados observados.

## Casos implementados y ejecutados

| ID | HU / RF | Objetivo y datos principales | Precondiciones y pasos | Resultado esperado | Estado / evidencia |
|---|---|---|---|---|---|
| CP-001 | HU-001 / RF-001 | Login correcto: vendedor1 / contraseña semilla de test. | Reiniciar store; POST login con credenciales de `tests/helpers.js`. | 200, token, rol VENDEDOR, sin `password` en perfil. | Aprobado — `tests/auth.test.js`, `test_login_exitoso`. |
| CP-002 | HU-001 / RF-001 | Rechazo de contraseña incorrecta. | Reiniciar store; POST login para vendedor1 con contraseña equivocada. | 401. | Aprobado — `tests/auth.test.js`, `test_login_credenciales_invalidas`. |
| CP-003 | HU-002 / RF-003 | Bloqueo de ruta protegida sin token. | Reiniciar store; GET `/api/v1/sorteos` sin Authorization. | 401. | Aprobado — `tests/auth.test.js`, `test_ruta_protegida_sin_token`. |
| CP-004 | HU-003 / RF-007 | Venta Chance de dos apuestas por 2000 y 1000. | Obtener token vendedor; sorteo semilla abierto; POST venta. | 201, código CH-, total 3000. | Aprobado — `tests/tickets.test.js`, `test_venta_chance`. |
| CP-005 | HU-004 / RF-008 | Venta Baloto con jugada manual y aleatoria. | Obtener token vendedor; sorteo Baloto abierto; POST venta. | 201 y total 11400. | Aprobado — `tests/tickets.test.js`, `test_venta_baloto`. |
| CP-006 | HU-003 / RF-007 | Rechazo de venta cerca del cierre. | Mover fecha del sorteo Chance a 5 min; vender. | 409. | Aprobado — `tests/tickets.test.js`, `test_venta_sorteo_cerrado`. |
| CP-007 | HU-003 / RF-007 | Impedir que ADMIN venda Chance. | Obtener token ADMIN; POST venta Chance. | 403. | Aprobado — `tests/tickets.test.js`, `test_admin_no_vende`. |
| CP-008 | HU-006 / RF-011 | Impedir que vendedor distinto anule tiquete ajeno y permitir al dueño. | Vender tiquete; intentar anular con vendedor2 y luego vendedor propietario, siempre con motivo. | 403 para ajeno; HTTP 200 y ANULADO para propietario. | Aprobado — `tests/tickets.test.js`, `test_anulacion_ticket`. |
| CP-009 | HU-006 / RF-011 | Rechazo de anulación fuera de tiempo. | Vender; retrotraer `creadoEn` 60 min; solicitar anulación. | 409. | Aprobado — `tests/tickets.test.js`, `test_anulacion_fuera_de_tiempo`. |
| CP-010 | HU-002/HU-005/HU-007 / RF-006/RF-010/RF-012 | Registrar resultado ganador, consultar ganador y pagar; rechazar pago duplicado. | Vender Chance ganador; publicar resultado; consultar y pagar dos veces. | Resultado 200; consulta ganador y premio 4500000; pago 200; segundo pago 409. | Aprobado — `tests/tickets.test.js`, `test_resultado_y_pago_premio`. |
| CP-011 | HU-003 / RF-007 | Premio directo Chance de cuatro cifras. | Validar apuesta `1234`, valor 1000; calcular con resultado ganador y no ganador. | Premio 4500000 en coincidencia y cero en no coincidencia. | Aprobado — `tests/chance.test.js`, `test_premio_directo_4_cifras`. |
| CP-012 | HU-003 / RF-007 | Premio directo por últimas tres y dos cifras. | Validar apuestas 234 y 34; calcular contra resultado 1234. | Premios 400000 y 50000. | Aprobado — `tests/chance.test.js`, `test_premio_directo_ultimas_cifras`. |
| CP-013 | HU-003 / RF-007 | Premio combinado dividido por permutaciones. | Validar combinado 123 por 600; calcular contra resultado 9321. | Premio 40000. | Aprobado — `tests/chance.test.js`, `test_premio_combinado_divide_por_permutaciones`. |
| CP-014 | HU-003 / RF-007 | Rechazo de números/modalidades/valores Chance inválidos. | Invocar validador con número alfabético, valor bajo/no múltiplo y combinado de dos cifras. | Cada caso lanza error. | Aprobado — `tests/chance.test.js`, `test_validacion_apuesta_invalida`. |
| CP-015 | HU-004 / RF-008 | Generación de jugada aleatoria Baloto. | Validar `{aleatoria:true}`. | Cinco números únicos y superbalota entre 1 y 16. | Aprobado — `tests/baloto.test.js`, `test_jugada_aleatoria_valida`. |
| CP-016 | HU-004 / RF-008 | Rechazo de repetidos y números fuera de rango Baloto. | Validar jugada con repetidos y jugada con número 44. | Ambos casos lanzan error. | Aprobado — `tests/baloto.test.js`, `test_jugada_numeros_repetidos`. |
| CP-017 | HU-004 / RF-008 | Cálculo de premios por aciertos Baloto. | Calcular combinaciones con 5+1, 4+1 y sin premio. | Acumulado, 2000000 y cero, respectivamente. | Aprobado — `tests/baloto.test.js`, `test_premio_por_aciertos`. |
| CP-018 | HU-008 / RF-013 | Resumen diario con ventas de Chance y Baloto. | Vender ambos productos; consultar resumen autenticado. | 200, dos tiquetes, ventas 5000/5700 y saldo 10700. | Aprobado — `tests/caja.test.js`, `test_resumen_caja_diario`. |
| CP-019 | HU-008 / RF-014 | Cierre de caja con faltante y rechazo de doble cierre. | Vender Chance por 5000; cerrar reportando 4000; repetir cierre. | 201, FALTANTE/-1000; segundo intento 409. | Aprobado — `tests/caja.test.js`, `test_cierre_caja_con_faltante`. |
| CP-020 | HU-002 / RF-005 | Crear sorteo Chance como ADMIN. | Obtener token ADMIN; POST sorteo con fecha futura. | 201 y estado ABIERTO. | Aprobado — `tests/caja.test.js`, `test_admin_crea_sorteo`. |

## Casos propuestos, pendientes de ejecución

| ID | HU / RF | Objetivo | Precondiciones / datos | Pasos resumidos | Resultado esperado | Estado |
|---|---|---|---|---|---|---|
| CP-021 | HU-001 / RF-002 | Perfil con token válido y rechazo de token expirado/inválido. | Usuarios semilla; token correcto y token alterado. | GET `/auth/perfil` con cada token. | 200 con solo campos del perfil; 401 en token inválido/expirado. | Pendiente de prueba |
| CP-022 | HU-002 / RF-003 | Filtros de lista por tipo y estado. | Sorteos semilla de ambos tipos. | GET `/sorteos?tipo=BALOTO&estado=ABIERTO`. | 200 con solo Baloto abierto. | Pendiente de prueba |
| CP-023 | HU-002 / RF-004 | Consultar sorteo existente e inexistente. | ID semilla 1 y un ID no existente. | GET `/sorteos/1` y `/sorteos/999`. | 200 en existente; 404 en inexistente. | Pendiente de prueba |
| CP-024 | HU-002 / RF-005 | Validar tipo, nombre, fecha inválida/pasada y permiso de rol. | ADMIN y vendedor; fechas inválidas/pasadas. | Intentar POST creación para cada variación. | Datos inválidos 400; vendedor 403; ADMIN válido 201. | Pendiente de prueba |
| CP-025 | HU-002 / RF-006 | Resultado de Baloto, resultado inválido y segundo resultado. | Crear/usar sorteo Baloto no jugado. | Publicar resultado; repetir; enviar números inválidos. | Resultado válido 200/liquidación; inválidos 400; repetición 409. | Pendiente de prueba |
| CP-026 | HU-003 / RF-007 | Límites de cantidad, apuesta y sorteo para venta Chance. | Token vendedor; casos con 0/6 apuestas, sorteo inexistente o tipo incorrecto. | Ejecutar POST de venta en cada caso. | Rechazo con 400/404; no insertar ticket inválido. | Pendiente de prueba |
| CP-027 | HU-004 / RF-008 | Límites de cantidad, rangos y autorización para venta Baloto. | Vendedor y ADMIN; jugada inválida, 0/11 jugadas. | Ejecutar solicitudes de venta. | Validación 400; ADMIN 403; sin tiquetes inválidos. | Pendiente de prueba |
| CP-028 | HU-005 / RF-009 | Filtros y aislamiento de vendedor al listar tickets. | Ventas de dos vendedores y tickets de ambos estados/fechas. | Consultar como vendedor y ADMIN con filtros. | Vendedor ve solo propios; ADMIN todos; filtros correctos. | Pendiente de prueba |
| CP-029 | HU-005 / RF-010 | Consultar código inexistente, no ganador y sorteo pendiente. | Tickets con distintos estados/resultado. | GET ticket por códigos y consultar respuesta. | 404 inexistente; `ganador` null antes de jugar y false para perdedor. | Pendiente de prueba |
| CP-030 | HU-006 / RF-011 | Validar motivo requerido, ticket ya anulado y sorteo jugado. | Preparar cada estado y vendedor/ADMIN. | Solicitar anulaciones con motivo ausente y tras cambio de estado. | 400 por motivo; 409 para estados no anulables. | Pendiente de prueba |
| CP-031 | HU-007 / RF-012 | Rechazar no ganador, sorteo no jugado y premio caducado. | Preparar tickets de cada condición; manipular fecha de jugado para caducidad. | POST `/tickets/:codigo/pagar`. | 409 antes de jugar/no ganador; 410 y estado CADUCADO tras vencimiento. | Pendiente de prueba |
| CP-032 | HU-009 / RF-015 | Verificar endpoint de salud. | App iniciada. | GET `/health` sin Authorization. | 200, `estado` igual a `ok`, `hora` ISO válida. | Pendiente de prueba |
| CP-033 | HU-008 / RF-013 | Resumen por fecha y consulta administrativa de otro vendedor. | Varios vendedores con ventas y pagos en fechas diferentes. | GET resumen con fecha; repetir como ADMIN con `vendedorId`. | Totales limitados a fecha y vendedor solicitado. | Pendiente de prueba |
| CP-034 | HU-008 / RF-014 | Cierre cuadrado/sobrante y monto inválido. | Caja abierta; reportar saldo exacto, excedente, negativo y decimal. | POST cierre con cada valor. | Estados CUADRADA/SOBRANTE; negativo/decimal rechazados con 400. | Pendiente de prueba |
| CP-035 | Todas / middleware | Manejo de JSON mal formado y ruta no existente. | Aplicación activa. | Enviar JSON inválido y método/ruta desconocidos. | 400 `JSON mal formado`; 404 para ruta desconocida. | Pendiente de prueba |

## Resultado de ejecución observado

Comando: `npm test` (`jest --runInBand`). En la última ejecución: 5 suites aprobadas, 20 pruebas aprobadas y 0 snapshots. La instalación requerida fue `npm ci`; no se instalaron dependencias de documentación. No se han ejecutado los casos CP-021 a CP-035.
