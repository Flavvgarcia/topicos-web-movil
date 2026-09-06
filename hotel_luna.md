# Análisis de Arquitectura: Hotel Luna

---

## Misión 1: Análisis de Problemas y Patrones

### Problema 1: Front Controller y Filtros redundantes
* **Descripción:** `FrontControllerServlet.java` crea un frontal y `FiltroCasero.java` una cadena paralela, aunque Spring MVC ya posee un frontal.
* **Capa:** Presentación / políticas transversales.
* **¿Por qué usar este patrón?:** Es *Front Controller* porque concentra el despacho HTTP; es *Chain* porque recorre filtros hasta que uno detiene la petición. No es simplemente *Facade*: no unifica una API de subsistemas, sino que recibe y despacha solicitudes.
* **Cuándo NO aplica:** No crear otro frontal si `DispatcherServlet` ya centraliza las rutas. Tampoco aplicar una cadena si solo existe una política local y no hay sucesión de manejadores.

### Problema 2: Lógica de negocio acoplada en el controlador
* **Descripción:** `ReservaCtrl.java` contiene todos los `if` de tipo de habitación y tarifa.
* **Capa:** Aplicación / dominio.
* **¿Por qué usar este patrón?:** Cada modalidad de tarifa puede encapsular una forma intercambiable de calcular el precio. No es *State*: la tarifa no representa el ciclo de vida de una reserva ni cambia por transiciones. Tampoco es *Factory*, porque el problema principal es variar el cálculo, no solamente crear objetos.
* **Cuándo NO aplica:** No extraería estrategias si hubiera dos reglas estables, triviales y sin probabilidad de crecer; en ese caso una función local podría ser suficiente.

### Problema 3: Persistencia directa sin abstracción ni transacción
* **Descripción:** El controlador ejecuta directamente tres `INSERT`/`UPDATE` independientes, sin transacción ni abstracción de persistencia.
* **Capa:** Datos / aplicación.
* **¿Por qué usar este patrón?:** El *Repository* separa el caso de uso de SQL y *Unit of Work* hace atómicas las escrituras relacionadas. No es *DAO* como solución completa: un DAO aislado no garantiza que reserva, habitación y cortesía se confirmen juntas.
* **Cuándo NO aplica:** No usar una unidad transaccional si la operación es solo lectura o si las escrituras pertenecen deliberadamente a sistemas con consistencia eventual.

### Problema 4: Integración acoplada para el pago
* **Descripción:** El pago se construye como XML y se interpreta buscando textos en `ReservaCtrl.java` mediante `RestTemplate`.
* **Capa:** Integración.
* **¿Por qué usar este patrón?:** El *Adapter* traduce la interfaz del banco a una operación de pago del dominio; la capa anticorrupción (*Anti-Corruption Layer*) evita que XML, códigos bancarios y estados externos contaminen el modelo interno. No es *Facade*: una fachada simplifica un subsistema, mientras aquí hay traducción de contratos y conceptos.
* **Cuándo NO aplica:** No aplicar una capa anticorrupción si ambos sistemas comparten exactamente el mismo contrato y modelo, o si la integración es temporal y de una sola línea sin reglas de traducción.

### Problema 5: Envío de correo fuertemente acoplado
* **Descripción:** El envío de correo está pegado al camino principal del cobro.
* **Capa:** Aplicación / integración.
* **¿Por qué usar este patrón?:** El caso de uso publica el evento “reserva pagada” y los notificadores reaccionan sin acoplar el cobro con el correo. No es *Unit of Work*: UoW coordina persistencia y transacción; *Observer* distribuye reacciones a un evento.
* **Cuándo NO aplica:** No usar *Observer* si solo existe una reacción fija, síncrona y crítica cuya respuesta debe formar parte explícita del resultado del caso de uso.

### Problema 6: Autenticación manual e insegura
* **Descripción:** La autenticación se implementa manualmente con sesión y cookies, y `HomeCtrl.java` confía en `admin_bypass`.
* **Capa:** Políticas transversales.
* **¿Por qué usar este patrón?:** La autenticación debe ejecutarse antes de los controladores y proteger todas las rutas. No es *Proxy* aplicado a un objeto de negocio: el problema ocurre en el ciclo HTTP completo, no solo al invocar un método.
* **Cuándo NO aplica:** No añadir una cadena propia si *Spring Security* ya ofrece autenticación, autorización, expiración de sesión y filtros configurables.

---

## Misión 2: Flujo de Ejecución de la Petición (Paso a Paso)

1. **Formulario:** El usuario pulsa el botón y el navegador envía `POST /reservar` con los parámetros.
2. **DispatcherServlet de Spring MVC:** Spring recibe centralmente la petición y evita que cada controlador tenga que gestionar el ciclo HTTP completo.
3. **Resolución de método (`@PostMapping`):** El `HandlerMapping` selecciona `ReservaCtrl.alta(...)` y Spring hace el *binding* de parámetros. No hay que inventar otro servicio de *routing*.
4. **`ReservaCtrl.alta`:** Orquesta el caso de uso. Actualmente también contiene la política de precios, lo que revela una mala separación de responsabilidades.
5. **Bloque de cálculo de precio:** Se decide el precio según el tipo y la tarifa. En una versión mantenible debería delegarse a un *Strategy* o servicio de precios.
6. **RestTemplate hacia `/Pay`:** Si el pago es con tarjeta, se envía XML al banco y se interpreta su respuesta. El cobro en mostrador evita esta integración.
7. **`JdbcTemplate.update`:** Se ejecutan operaciones sobre las tablas `reservas`, `habitaciones` y `spa_cortesia` para persistir la reserva pagada, ocupar la habitación y registrar la cortesía. Actualmente no hay servicio ni transacción explícita.
8. **`JavaMailSender`:** Se envía el correo después de las escrituras. Actualmente un fallo del correo puede dejar ambiguo el resultado del cobro y la persistencia.
9. **Redirección (`redirect:/`):** El controlador devuelve una redirección para evitar el reenvío accidental del formulario al recargar la página.

---

## Misión 3: Decisiones Arquitectónicas

### 1. ¿Cuándo corresponde usar un Front Controller?
Corresponde cuando muchas peticiones HTTP necesitan un punto común para:
* Resolver rutas.
* Aplicar autenticación y autorización.
* Manejar errores globales.
* Establecer el contexto de la petición.
* Delegar al controlador adecuado.

### 2. Orden recomendado de la cadena (Middlewares/Filtros)
Un orden razonable para el flujo es:
1. **Autenticación y expiración de sesión:** Si la identidad no es válida, la petición termina antes del cobro o cualquier otra acción sensible.
2. **Comprobación de cupo:** Debe comprobarse antes de cobrar o confirmar la inscripción. Sin embargo, el cupo es una regla de negocio, no una política transversal pura; lo correcto es que la evalúe el servicio del caso de uso, protegido por transacción y concurrencia.
3. **Bitácora de la petición aceptada:** Puede registrar al usuario, la operación y un identificador de correlación antes de delegar al caso de uso; también debe registrar el resultado o error después.
4. **Caso de uso (“pagar inscripción” / “reservar”):** El servicio valida las reglas, realiza el pago, persiste y publica el evento de confirmación.