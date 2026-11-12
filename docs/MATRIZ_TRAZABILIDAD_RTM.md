# Matriz de trazabilidad de requisitos (RTM)

La relación RNF indica la cualidad relevante, no que se haya demostrado un nivel de servicio. `CP-...` remite a [CASOS_DE_PRUEBA.md](CASOS_DE_PRUEBA.md). Los estados se basan en la ejecución de Jest efectuada el 12-11-2026: 20/20 pruebas aprobadas; los casos restantes están propuestos y no ejecutados.

| ID HU | Historia de usuario | ID RF | ID RNF | Módulo / Controlador | Archivo / Función | Caso de prueba | Estado QA |
|---|---|---|---|---|---|---|---|
| HU-001 | Autenticarse y consultar el perfil | RF-001 | RNF-001, RNF-002 | Autenticación / auth | `src/services/authService.js` — `login`; `src/controllers/authController.js` | CP-001, CP-002 | Aprobado en suite: login correcto e inválido |
| HU-001 | Autenticarse y consultar el perfil | RF-002 | RNF-001, RNF-002 | Autenticación / auth | `src/middlewares/auth.js` — `autenticar`; `src/services/authService.js` — `perfil` | CP-021 | Pendiente de prueba |
| HU-002 | Consultar y administrar sorteos | RF-003 | RNF-001, RNF-010 | Sorteos / sorteos | `src/services/sorteoService.js` — `listar` | CP-003, CP-022 | CP-003 aprobado (middleware sin token); contenido/filtros pendiente |
| HU-002 | Consultar y administrar sorteos | RF-004 | RNF-001, RNF-010 | Sorteos / sorteos | `src/services/sorteoService.js` — `obtener` | CP-023 | Pendiente de prueba |
| HU-002 | Consultar y administrar sorteos | RF-005 | RNF-001, RNF-010 | Sorteos / sorteos | `src/services/sorteoService.js` — `crear` | CP-020, CP-024 | CP-020 aprobado (creación); validaciones pendientes |
| HU-002 | Consultar y administrar sorteos | RF-006 | RNF-001, RNF-010 | Sorteos / sorteos | `src/services/sorteoService.js` — `registrarResultado`, `liquidarTicket` | CP-010, CP-025 | CP-010 aprobado (Chance y pago); resultado Baloto pendiente |
| HU-003 | Vender un tiquete de Chance | RF-007 | RNF-001, RNF-010 | Tiquetes / tickets | `src/services/ticketService.js` — `venderChance`; `src/services/chanceService.js` — `validarApuesta` | CP-004, CP-006, CP-007, CP-011, CP-012, CP-013, CP-014, CP-026 | CP-004/006/007/011/012/013/014 aprobados; validaciones HTTP adicionales pendientes |
| HU-004 | Vender un tiquete de Baloto | RF-008 | RNF-001, RNF-010 | Tiquetes / tickets | `src/services/ticketService.js` — `venderBaloto`; `src/services/balotoService.js` — `validarJugada` | CP-005, CP-015, CP-016, CP-017, CP-027 | CP-005/015/016/017 aprobados; permisos/validaciones HTTP pendientes |
| HU-005 | Listar y consultar tiquetes | RF-009 | RNF-001, RNF-010 | Tiquetes / tickets | `src/services/ticketService.js` — `listar` | CP-028 | Pendiente de prueba |
| HU-005 | Listar y consultar tiquetes | RF-010 | RNF-001, RNF-010 | Tiquetes / tickets | `src/services/ticketService.js` — `consultar` | CP-010, CP-029 | CP-010 aprobado (consulta ganador); no encontrado pendiente |
| HU-006 | Anular un tiquete | RF-011 | RNF-001, RNF-010 | Tiquetes / tickets | `src/services/ticketService.js` — `anular` | CP-008, CP-009, CP-030 | CP-008/009 aprobados; límites adicionales pendientes |
| HU-007 | Pagar un premio | RF-012 | RNF-001, RNF-010 | Tiquetes / tickets | `src/services/ticketService.js` — `pagarPremio` | CP-010, CP-031 | CP-010 aprobado (pago y duplicado); caducidad pendiente |
| HU-008 | Consultar y cerrar caja | RF-013 | RNF-001, RNF-010 | Caja / caja | `src/services/cajaService.js` — `resumen` | CP-018 | Aprobado en suite (resumen de dos ventas) |
| HU-008 | Consultar y cerrar caja | RF-014 | RNF-001, RNF-010 | Caja / caja | `src/services/cajaService.js` — `cerrar` | CP-019 | Aprobado en suite (faltante y cierre duplicado) |
| HU-009 | Verificar disponibilidad HTTP | RF-015 | RNF-005, RNF-006 | Aplicación HTTP | `src/app.js` — `GET /health` | CP-032 | Pendiente de prueba |

## Comprobaciones de consistencia

- HU-001 a HU-009 tienen al menos un RF relacionado.
- RF-001 a RF-015 referencian rutas, controladores o funciones presentes en el código analizado.
- Cada RF se relaciona con uno o más RNF identificados en [REQUISITOS_NO_FUNCIONALES.md](REQUISITOS_NO_FUNCIONALES.md); sus estados distinguen lo probado, pendiente y no identificado.
- Cada fila remite a casos existentes o propuestos; no se considera aprobado un caso propuesto.
