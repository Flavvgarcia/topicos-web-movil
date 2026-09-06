# Análisis de Arquitectura: Sabores Laravel

---

## Misión 1: Análisis de Problemas y Patrones

### Problema 1: Autenticación manual e inconsistente
* **Descripción:** La autenticación se comprueba manualmente en controladores mediante sesión y cookies, y algunas rutas ni siquiera usan middleware.
* **Capa:** Políticas transversales.
* **¿Por qué usar este patrón?:** La autenticación debe envolver la petición antes de llegar al caso de uso (Middleware / Filtro). No es *Front Controller*: este centraliza la entrada HTTP, pero no debería contener cada política concreta.
* **Cuándo NO aplica:** No aplicaría para una operación pública que no requiera identidad ni autorización.

### Problema 2: Consultas SQL directas en las vistas
* **Descripción:** `menu.blade.php` abre una conexión `mysqli` y ejecuta SQL directamente desde la vista.
* **Capa:** Datos / Presentación.
* **¿Por qué usar este patrón?:** El *Repository* encapsula la consulta y ofrece datos a la aplicación. No es *Facade*: no se busca simplificar varios subsistemas, sino separar la persistencia de la vista.
* **Cuándo NO aplica:** No sería necesario para una prueba desechable con una única consulta, aunque seguiría siendo mala práctica en una aplicación mantenida.

### Problema 3: Controlador monolítico (Lógica acoplada)
* **Descripción:** `PedidoController::guardar` calcula precios, interpreta el pago, modifica sesión de negocio, persiste datos y decide el flujo completo.
* **Capa:** Aplicación / Dominio.
* **¿Por qué usar este patrón?:** Un *Servicio de Aplicación* coordina el caso de uso “crear y pagar pedido”. No es *Facade*: una fachada simplifica el acceso a un subsistema, mientras aquí hay reglas y coordinación de negocio.
* **Cuándo NO aplica:** No aplicaría si el controlador solo adaptara la petición y el caso de uso fuera trivial y sin reglas.

### Problema 4: Integración de pagos acoplada y manual
* **Descripción:** El pago con tarjeta construye XML y llama directamente a una URL externa mediante `curl`.
* **Capa:** Integración.
* **¿Por qué usar este patrón?:** El *Adapter* traduce el contrato interno de pago al protocolo XML de la pasarela. No es *Facade*: no solo oculta varias operaciones; transforma interfaces y formatos incompatibles.
* **Cuándo NO aplica:** No aplicaría si la pasarela ya expusiera exactamente el contrato que usa la aplicación o si no existiera un sistema externo.

### Problema 5: Falta de transaccionalidad en múltiples escrituras
* **Descripción:** Se insertan el pedido, el detalle y la cola de reparto en operaciones separadas, sin transacción.
* **Capa:** Datos / Aplicación.
* **¿Por qué usar este patrón?:** El *Unit of Work* agrupa cambios relacionados y los confirma o revierte atómicamente. No es *Observer*: un observador notifica acontecimientos, pero no garantiza consistencia entre varias escrituras.
* **Cuándo NO aplica:** No aplicaría para una sola escritura independiente o para datos deliberadamente eventuales.

### Problema 6: Envío de correos síncrono desde el controlador
* **Descripción:** El controlador envía correos directamente después de pagar o cerrar un reparto.
* **Capa:** Aplicación / Integración.
* **¿Por qué usar este patrón?:** Publicar un evento (`PedidoPagado` o `PedidoEntregado`) y usar un *Observer* permitiría desacoplar la notificación del caso de uso. No es *Unit of Work*: el correo es una reacción, no una unidad de persistencia.
* **Cuándo NO aplica:** No aplicaría si el envío síncrono fuera parte estricta del resultado y la operación debiera fallar cuando el correo no se envía.

---

## Misión 2: Flujo de Ejecución de la Petición (Paso a Paso)

1. **Vista HTML:** `menu.blade.php` genera un formulario que envía `platillo_id`, envío, forma de pago y datos del cliente.
2. **Front Controller nativo de Laravel:** La petición entra por el punto único del framework, no por un frontal creado manualmente en este proyecto.
3. **Router de Laravel:** En `web.php`, `POST /pedir` se asocia con `PedidoController::guardar`.
4. **Middleware de grupo web:** Se inicia la sesión mediante `StartSession`. Sin embargo, la ruta `/pedir` no tiene aplicado el alias `auth`.
5. **Controlador:** `PedidoController::guardar` lee la petición. Aquí debería invocarse un servicio de aplicación, pero actualmente el controlador contiene el caso de uso completo.
6. **Consulta de precios:** Se consultan los platillos directamente con SQL, sin utilizar un *Repository*.
7. **Cálculo del total:** Se suman los platillos y el coste de envío dentro del controlador. No hay una política o servicio de dominio separado.
8. **Pago (Estrategias de cobro):**
   * *Efectivo:* Se marca como correcto localmente.
   * *Vale de despensa:* Se inserta directamente en `vales_usados`.
   * *Puntos:* Se actualizan directamente los puntos del usuario.
   * *Tarjeta o Débito:* Se construye XML y se llama directamente a la pasarela con `curl`; no existe un *Adapter* formal.
9. **Persistencia del pedido:** Se insertan registros en `pedidos`, `pedido_detalle` y `cola_reparto`. No existe *Unit of Work* ni transacción de base de datos.
10. **Notificación:** Se ejecuta la función `mail(...)` directamente. No hay eventos ni *Observer*.
11. **Respuesta HTTP:** Se devuelve un script de JavaScript que muestra un mensaje y redirige al inicio.

---

## Misión 3: Decisiones Arquitectónicas

### 1. ¿Cuándo corresponde usar un Front Controller?
Corresponde cuando una aplicación web necesita un único punto de entrada para centralizar:
* Creación de la petición.
* Selección del router.
* Manejo común de errores.
* Carga del framework.
* Ejecución de middleware.
* Generación de la respuesta.

**Contexto en Laravel:** Laravel ya proporciona ese Front Controller de manera nativa (`public/index.php`). Por lo tanto, no debe crearse otro controlador frontal personalizado dentro del directorio `app/` para intentar reemplazarlo.

### 2. Orden de la cadena (Middlewares/Filtros) recomendado
La composición correcta para el flujo de la solicitud sería:
1. **Autenticación** (Verificar identidad del usuario)
2. ➔ **Verificación de cupo** (Regla de negocio preliminar)
3. ➔ **Bitácora** (Registrar intento/acción)
4. ➔ **Caso de uso** (Inscripción, cobro o acción principal)