# Puesto Chance API

[![Node.js](https://img.shields.io/badge/Node.js-CommonJS-339933?logo=node.js&logoColor=white)](https://nodejs.org/)
[![Express](https://img.shields.io/badge/Express-4.x-000000?logo=express&logoColor=white)](https://expressjs.com/)
[![Jest](https://img.shields.io/badge/tests-Jest-C21325?logo=jest&logoColor=white)](https://jestjs.io/)
[![API](https://img.shields.io/badge/API-REST%20%7C%20JSON-blue)](docs/DOCUMENTACION_API.md)

## Descripción y objetivo

API REST de demostración para la operación de un puesto de chance. El código implementa autenticación con JWT y roles, administración de sorteos, ventas de Chance y Baloto, consulta/anulación/pago de tiquetes y resumen/cierre de caja.

Su objetivo, reconstruido a partir del código, es ofrecer operaciones HTTP para esos flujos de venta y control. Los datos se guardan en memoria y se reinician al reiniciar el proceso: no hay conexión a una base de datos persistente.

## Tecnologías

- JavaScript (CommonJS) y Node.js.
- Express 4, CORS, Morgan y dotenv.
- JWT (`jsonwebtoken`) y hash de contraseñas (`bcryptjs`).
- Jest 29 y Supertest para pruebas.

## Arquitectura y estructura

La aplicación separa la configuración y el servidor (`src/config`, `src/server.js`), el ensamblaje HTTP (`src/app.js`), las rutas (`src/routes`), los controladores (`src/controllers`), las reglas de negocio (`src/services`), la autenticación y errores (`src/middlewares`), el almacenamiento en memoria (`src/data`) y utilidades (`src/utils`). Las pruebas están en `tests/`.

```text
src/
  app.js, server.js
  config/
  controllers/
  data/
  middlewares/
  routes/
  services/
  utils/
tests/
docs/
```

## Requisitos previos

- Node.js y npm instalados. El repositorio no declara una versión mínima; la versión usada para verificar esta entrega fue Node.js 24.16.0 y npm 11.13.0.
- Git para clonar el repositorio.

## Instalación y configuración

Desde la raíz del proyecto:

```sh
npm ci
```

Copie `.env.example` como `.env` y configure `JWT_SECRET` con un valor privado y aleatorio:

```powershell
Copy-Item .env.example .env
```

Variables leídas por `src/config/index.js`:

| Variable | Predeterminado | Uso |
|---|---:|---|
| `PORT` | `3000` | Puerto HTTP |
| `JWT_SECRET` | `cambiar-este-secreto` en el código | Firma de tokens. Debe reemplazarse en cualquier entorno no desechable. |
| `JWT_EXPIRES_IN` | `8h` | Vigencia del JWT |
| `ZONA_HORARIA` | `America/Bogota` | Cálculo de fecha local |
| `MINUTOS_ANULACION` | `10` | Ventana de anulación para vendedor |
| `MINUTOS_CIERRE_VENTAS` | `15` | Anticipación de cierre de ventas |
| `APUESTA_MIN_CHANCE` | `500` | Apuesta mínima de Chance |
| `APUESTA_MAX_CHANCE` | `50000` | Apuesta máxima de Chance |
| `VALOR_BALOTO` | `5700` | Precio por jugada de Baloto |
| `DIAS_CADUCIDAD_PREMIO` | `365` | Vigencia del premio después del sorteo |

El repositorio incluye usuarios y datos semilla definidos en `src/data/store.js`; son datos locales de demostración, no una gestión de usuarios persistente. No use las credenciales semilla ni el secreto predeterminado en producción. El ejemplo de entorno no contiene credenciales.

## Ejecución

Modo de desarrollo (reinicio automático con nodemon):

```sh
npm run dev
```

Ejecución normal:

```sh
npm start
```

La API escucha en `http://localhost:3000` por defecto. La comprobación de salud está disponible en `GET http://localhost:3000/health`.

## API y OpenAPI

Las rutas de negocio usan el prefijo `/api/v1`. La descripción completa de rutas, datos, autenticación y respuestas está en [docs/DOCUMENTACION_API.md](docs/DOCUMENTACION_API.md); la especificación OpenAPI 3.1 está en [docs/openapi.yaml](docs/openapi.yaml).

El proyecto no incluye Swagger UI ni un servidor de documentación interactiva. La especificación YAML se puede abrir en una herramienta compatible con OpenAPI; no existe una ruta `/docs` implementada.

## Pruebas

```sh
npm test
```

Jest ejecuta los archivos `tests/*.test.js` en serie. En la validación de esta entrega pasaron 5 suites y 20 pruebas. El alcance real de las pruebas y los escenarios propuestos aún pendientes se detallan en [docs/CASOS_DE_PRUEBA.md](docs/CASOS_DE_PRUEBA.md).

## Autores

El repositorio no identifica autores en un archivo de metadatos o licencia; consulte el historial Git para atribución.

## Licencia

No se encontró un archivo de licencia ni una declaración de licencia en el manifiesto. Por tanto, no se atribuye una licencia en esta documentación.

## Documentación de ingeniería inversa

- [Historias de usuario](docs/HISTORIAS_USUARIO.md)
- [Requisitos funcionales](docs/REQUISITOS_FUNCIONALES.md)
- [Requisitos no funcionales](docs/REQUISITOS_NO_FUNCIONALES.md)
- [Matriz de trazabilidad RTM](docs/MATRIZ_TRAZABILIDAD_RTM.md)
- [Documentación de API y funciones](docs/DOCUMENTACION_API.md)
- [Casos de prueba](docs/CASOS_DE_PRUEBA.md)
- [Proceso y alcance de ingeniería inversa](docs/INGENIERIA_INVERSA.md)
