# Historias de usuario reconstruidas

Estas historias describen comportamientos observables en rutas, controladores, servicios y pruebas. No se consideran requisitos originales aprobados por un propietario del producto.

## HU-001 — Autenticarse y consultar el perfil

- **Como:** usuario registrado (administrador o vendedor).
- **Quiero:** iniciar sesión y consultar mi perfil autenticado.
- **Para:** obtener un token y conocer la identidad/rol asociado a mis operaciones.
- **Criterios de aceptación:**
  1. `POST /api/v1/auth/login` recibe `usuario` y `password`; con credenciales válidas devuelve token y perfil sin contraseña.
  2. Credenciales incorrectas producen HTTP 401; faltantes producen HTTP 400.
  3. `GET /api/v1/auth/perfil` requiere `Bearer` válido y devuelve id, usuario, nombre y rol.
- **Prioridad:** Alta (prerrequisito de las rutas protegidas).
- **Módulo relacionado:** autenticación.

## HU-002 — Consultar y administrar sorteos

- **Como:** administrador para crear y publicar resultados; usuario autenticado para consultar sorteos.
- **Quiero:** listar/consultar sorteos, crear uno futuro y registrar su resultado.
- **Para:** habilitar ventas y liquidar los tiquetes asociados.
- **Criterios de aceptación:**
  1. Se pueden filtrar sorteos por `tipo` y `estado` y consultar uno por id.
  2. Solo ADMIN puede crear sorteos CHANCE o BALOTO con nombre y fecha futura válida.
  3. Solo ADMIN puede registrar un resultado válido una vez; el sorteo pasa a JUGADO y se liquidan sus tiquetes activos.
- **Prioridad:** Alta.
- **Módulo relacionado:** sorteos.

## HU-003 — Vender un tiquete de Chance

- **Como:** vendedor autenticado.
- **Quiero:** registrar de una a cinco apuestas de Chance en un sorteo abierto.
- **Para:** emitir un tiquete y registrar el valor vendido.
- **Criterios de aceptación:**
  1. Solo el rol VENDEDOR puede vender Chance.
  2. El sorteo debe ser CHANCE, estar ABIERTO y conservar al menos los minutos de cierre configurados.
  3. Las apuestas cumplen dígitos, modalidad y límites/múltiplos de valor configurados; el total es la suma.
  4. La respuesta HTTP 201 incluye código `CH-...` y estado ACTIVO.
- **Prioridad:** Alta.
- **Módulo relacionado:** tiquetes / Chance.

## HU-004 — Vender un tiquete de Baloto

- **Como:** vendedor autenticado.
- **Quiero:** registrar de una a diez jugadas de Baloto, manuales o aleatorias.
- **Para:** emitir un tiquete y registrar el valor correspondiente por jugada.
- **Criterios de aceptación:**
  1. Solo VENDEDOR puede vender en un sorteo BALOTO abierto y dentro de la ventana.
  2. Cada jugada manual contiene cinco números únicos entre 1 y 43 y superbalota entre 1 y 16; la modalidad aleatoria genera valores en esos rangos.
  3. El total corresponde al número de jugadas por `VALOR_BALOTO`; la respuesta HTTP 201 contiene código `BL-...` y estado ACTIVO.
- **Prioridad:** Alta.
- **Módulo relacionado:** tiquetes / Baloto.

## HU-005 — Listar y consultar tiquetes

- **Como:** usuario autenticado.
- **Quiero:** listar tiquetes con filtros y consultar uno por código.
- **Para:** verificar su estado, sorteo, resultado y condición de ganador.
- **Criterios de aceptación:**
  1. La lista acepta filtros `fecha` y `estado`; ADMIN puede listar todos y cada vendedor solo los suyos.
  2. La consulta por código añade datos resumidos del sorteo y `ganador` cuando el sorteo ya fue jugado.
  3. Un código inexistente produce HTTP 404.
- **Prioridad:** Alta.
- **Módulo relacionado:** tiquetes.

## HU-006 — Anular un tiquete

- **Como:** vendedor propietario del tiquete o administrador.
- **Quiero:** anular un tiquete activo indicando un motivo.
- **Para:** corregir una venta errónea antes de que el sorteo se juegue.
- **Criterios de aceptación:**
  1. Un vendedor solo anula sus tiquetes y dentro de la ventana configurada; ADMIN no está sujeto a esa ventana.
  2. Se exige motivo; no se anula un tiquete no ACTIVO ni de sorteo JUGADO.
  3. La anulación conserva usuario, motivo y fecha y cambia el estado a ANULADO.
- **Prioridad:** Media.
- **Módulo relacionado:** tiquetes.

## HU-007 — Pagar un premio

- **Como:** vendedor autorizado.
- **Quiero:** registrar el pago de un premio de un tiquete ganador vigente.
- **Para:** dejar constancia del desembolso y evitar pagos duplicados.
- **Criterios de aceptación:**
  1. El sorteo debe estar JUGADO, el tiquete ACTIVO y tener premio mayor que cero.
  2. Si el premio superó su vigencia configurada, el tiquete se marca CADUCADO y no se paga.
  3. El pago crea un registro asociado y cambia el tiquete a PAGADO; un segundo pago se rechaza.
- **Prioridad:** Alta.
- **Módulo relacionado:** tiquetes / pagos.

## HU-008 — Consultar y cerrar caja

- **Como:** vendedor autenticado.
- **Quiero:** consultar el resumen diario y cerrar caja reportando el efectivo.
- **Para:** comparar ventas y premios pagados con el efectivo disponible.
- **Criterios de aceptación:**
  1. El resumen calcula ventas, anulaciones, pagos, saldo esperado y si la caja ya está cerrada para la fecha.
  2. El cierre requiere efectivo entero no negativo y solo puede registrarse una vez por vendedor y fecha.
  3. El cierre conserva diferencia y estado CUADRADA, SOBRANTE o FALTANTE.
- **Prioridad:** Alta.
- **Módulo relacionado:** caja.

## HU-009 — Verificar disponibilidad HTTP

- **Como:** cliente de integración u operador.
- **Quiero:** consultar el endpoint de salud.
- **Para:** confirmar que el proceso responde HTTP.
- **Criterios de aceptación:**
  1. `GET /health` no requiere token y devuelve `estado: "ok"` y una hora ISO.
- **Prioridad:** Media.
- **Módulo relacionado:** aplicación HTTP.
