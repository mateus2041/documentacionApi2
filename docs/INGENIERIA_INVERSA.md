# Ingeniería inversa — alcance y hallazgos

## Propósito

Reconstruir la documentación funcional y técnica a partir del código disponible, sin afirmar requisitos originales no suministrados ni capacidades ausentes.

## Elementos analizados

- Manifiesto y dependencias: `package.json`, `package-lock.json`.
- Ensamblaje y ejecución HTTP: `src/app.js`, `src/server.js`, `src/routes/index.js`.
- Configuración y datos: `src/config/index.js`, `src/data/store.js`.
- Controladores: `src/controllers/authController.js`, `sorteoController.js`, `ticketController.js`, `cajaController.js`.
- Servicios: `authService.js`, `sorteoService.js`, `ticketService.js`, `chanceService.js`, `balotoService.js`, `cajaService.js`.
- Soporte transversal: `src/middlewares/auth.js`, `errorHandler.js`; `src/utils/AppError.js`, `fechas.js`.
- Calidad existente: `tests/auth.test.js`, `tickets.test.js`, `chance.test.js`, `baloto.test.js`, `caja.test.js`, `helpers.js`.

## Método

1. Inventariar estructura, manifiesto, configuración, rutas y pruebas.
2. Seguir cada ruta desde middleware y controlador al servicio que contiene las reglas.
3. Extraer actores/roles, datos, validaciones, transiciones de estado y respuestas observables.
4. Asociar esas evidencias a HU y RF; clasificar RNF separando código comprobado, ejecución de pruebas y carencias.
5. Relacionar requisitos con pruebas existentes o casos propuestos y revisar consistencia de identificadores.
6. Contrastar la documentación API con las rutas declaradas.

## Arquitectura y funcionamiento observado

Express inicializa CORS y JSON, registra Morgan fuera del entorno `test`, expone `/health` y monta rutas bajo `/api/v1`. Los controladores delegan reglas en servicios. `autenticar` verifica JWT y existencia/estado del usuario; `permitir` restringe por rol. `errorHandler` traduce errores de aplicación y errores HTTP.

`src/data/store.js` crea en memoria usuarios, sorteos y secuencias; tickets, pagos y cierres empiezan vacíos. No existe en este repositorio esquema SQL/NoSQL, ORM, driver de base de datos ni persistencia durable. Al reiniciar el proceso, el estado operativo se pierde.

## Reglas y límites identificados

- Roles semilla: ADMIN y VENDEDOR; rutas y permisos definidos en `src/routes/index.js`.
- Chance: números de 1–4 dígitos, DIRECTO/COMBINADO, valor entero dentro del mínimo/máximo y múltiplo de 100; de 1 a 5 apuestas por tiquete; tabla de premios en `chanceService.js`.
- Baloto: cinco números únicos de 1–43, superbalota 1–16, jugadas aleatorias o manuales; de 1 a 10 jugadas por tiquete; precio y tabla de premios en `balotoService.js`/configuración.
- Venta: sorteo del tipo esperado, ABIERTO y antes de la ventana de cierre configurada.
- Sorteo: tipo permitido, fecha futura; resultado cambia estado a JUGADO y liquida tiquetes activos.
- Anulación: motivo requerido; propietario vendedor o ADMIN; vendedor limitado por ventana; no anula sorteo jugado.
- Pago: solo tiquete ACTIVO con premio tras jugarse el sorteo; marca CADUCADO fuera de vigencia; evita pago repetido.
- Caja: suma tickets no anulados y pagos; cierre único por vendedor/fecha y diferencia contra efectivo reportado.

## Rutas y documentación de API

Se identificaron 14 rutas bajo `/api/v1` y una ruta de salud (`/health`). No se encontraron dependencias ni configuración preexistente de Swagger/OpenAPI. Para satisfacer la documentación reproducible se agregó una especificación OpenAPI 3.1 estática en `openapi.yaml`; no se añadió UI ni endpoint de Swagger.

## Calidad y limitaciones

La suite presente fue ejecutada con `npm test`: 5 suites y 20 pruebas aprobadas. Existe cobertura para autenticación básica, venta, algunas reglas de juego, anulación, resultado/pago y caja; varias rutas y ramas no tienen prueba directa. Los casos adicionales están marcados como pendientes en `CASOS_DE_PRUEBA.md`.

Riesgos/pendientes observables:

- secreto JWT de respaldo conocido en configuración;
- datos semilla con contraseñas de demostración en código y sin gestión persistente;
- inexistencia de persistencia, transacciones, monitoreo de disponibilidad y mediciones de rendimiento;
- versión soportada de Node.js no declarada;
- no se identificaron autores ni archivo de licencia;
- la instalación de dependencias informó 8 avisos de auditoría (6 de severidad baja y 2 moderada); no se ejecutó `npm audit fix` ni se alteraron dependencias.

## Resultado de reconstrucción

- 9 historias de usuario (HU-001 a HU-009).
- 15 requisitos funcionales (RF-001 a RF-015).
- 10 observaciones/requisitos no funcionales (RNF-001 a RNF-010).
- 20 casos automatizados existentes ejecutados y aprobados; 15 casos adicionales propuestos pendientes.
- 14 endpoints de negocio y 1 endpoint de salud documentados.

Los hallazgos describen el estado observado, no equivalen a aprobación formal de requisitos por cliente/propietario ni a certificación de seguridad, rendimiento o disponibilidad.
