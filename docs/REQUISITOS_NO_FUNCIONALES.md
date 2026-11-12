# Requisitos no funcionales y evidencia

No se deduce cumplimiento por ausencia de fallos. “Implementado y comprobado” se limita a comportamientos cubiertos por pruebas ejecutadas; “implementado, pendiente de pruebas” describe código observable sin prueba automatizada directa; “no identificado” indica que el repositorio no establece ni implementa evidencia suficiente. Las recomendaciones no son requisitos actuales.

| ID | Categoría | Requisito/observación reconstruida | Evidencia | Estado |
|---|---|---|---|---|
| RNF-001 | Seguridad | Las rutas protegidas exigen JWT Bearer válido y algunas operaciones restringen roles. | `src/middlewares/auth.js`, rutas; pruebas de token ausente, rol administrador no vendedor y vendedor ajeno. No hay prueba de expiración/firmas erróneas. | **Implementado y comprobado parcialmente** |
| RNF-002 | Seguridad | Las contraseñas semilla se almacenan como hashes bcrypt y no se incluyen en el perfil devuelto. | `src/data/store.js`, `src/services/authService.js`; `tests/auth.test.js` verifica que la respuesta no exponga `password`. | **Implementado y comprobado parcialmente** |
| RNF-003 | Seguridad | Existe un secreto JWT predeterminado conocido en el código; no hay validación de fortaleza/presencia al iniciar. | `src/config/index.js`: valor de respaldo `cambiar-este-secreto`. | **Riesgo observado; requiere corregirse/configurarse antes de uso real** |
| RNF-004 | Rendimiento | No se identifican objetivos, mediciones, límites de carga, caché ni pruebas de rendimiento. | No hay herramienta ni configuración de benchmark en manifiesto/código. | **No identificado en el código** |
| RNF-005 | Disponibilidad | No se identifican redundancia, persistencia, recuperación, health checks externos ni SLA. `GET /health` solo confirma respuesta HTTP del proceso. | `src/app.js`, `src/data/store.js`. | **Parcialmente implementado; disponibilidad no comprobada** |
| RNF-006 | Usabilidad | La interfaz identificada es una API JSON; no se encontró interfaz web, guía de errores interactiva ni requisitos de accesibilidad. | Express JSON en `src/app.js`; OpenAPI en esta entrega es documentación estática, no UI ejecutable. | **API implementada; usabilidad de consumidor pendiente de evaluación** |
| RNF-007 | Mantenibilidad | El código separa rutas, controladores, servicios, middleware, configuración y utilidades; Jest/Supertest prueba reglas y flujos. | Estructura `src/`, `tests/`; suite ejecutada: 5 suites, 20 pruebas aprobadas. | **Implementado y comprobado parcialmente** |
| RNF-008 | Escalabilidad | No hay almacenamiento compartido o persistente: los datos viven en memoria y `store.reset()` resemilla; el proceso aislado no sincroniza instancias. | `src/data/store.js`. | **Limitación comprobada por implementación; escalado horizontal no identificado** |
| RNF-009 | Compatibilidad | El proyecto usa CommonJS y el manifiesto no fija rango de Node.js; las pruebas se ejecutaron con Node.js 24.16.0/npm 11.13.0. | `package.json`; verificación local de esta entrega. | **Ejecución comprobada en una versión; rango compatible pendiente de definir** |
| RNF-010 | Integridad de datos | Se validan formatos/rangos de apuestas y transiciones de estado para venta, anulación, sorteo y pago; no hay transacciones ni persistencia durable. | `src/services/*Service.js`; pruebas unitarias de Chance/Baloto y flujos de tiquetes/caja. | **Validaciones implementadas y parcialmente comprobadas; durabilidad no implementada** |

## Recomendaciones (no son capacidades actuales)

1. Rechazar el inicio si `JWT_SECRET` falta o conserva el valor predeterminado.
2. Sustituir almacenamiento en memoria por persistencia transaccional si se requiere conservar operaciones o ejecutar más de una instancia.
3. Definir versiones de Node.js soportadas, límites de rendimiento, monitoreo y objetivos de disponibilidad.
4. Ampliar pruebas de autorización, validación de entradas, caducidad, filtros y condiciones límite.
