# Análisis de Arquitectura: TallerPro Express

---

## Misión 1: Análisis de Problemas y Patrones

### Problema 1: Implementación manual de la cadena de autenticación y bitácora
* **Descripción:** La autenticación y la bitácora se implementan como clases enlazadas manualmente, aunque Express ya tiene una cadena de middleware nativa.
* **Capa:** Políticas transversales.
* **¿Por qué usar este patrón?:** El *Middleware* ejecuta lógica antes o después de la ruta; *Chain of Responsibility* decide qué objeto continúa procesando una petición. Aquí no hay selección de sucesor: siempre se ejecutan autenticación, bitácora y cobro.
* **Cuándo NO aplica:** No aplicaría *Middleware* para una regla exclusiva de un único método de dominio que no cruza peticiones HTTP.

### Problema 2: Controlador abrumado con lógica múltiple (`CobroOrden`)
* **Descripción:** `CobroOrden` calcula precios, decide el medio de pago, actualiza varias tablas y envía correos.
* **Capa:** Aplicación / Dominio.
* **¿Por qué usar este patrón?:** Un *Servicio de Aplicación* coordina un caso de uso; no es una *Facade*, porque no solo simplifica una API de subsistemas, sino que contiene reglas y efectos de negocio.
* **Cuándo NO aplica:** No aplicaría un servicio de aplicación si el endpoint solo transforma una entrada y llama a una operación atómica ya encapsulada.

### Problema 3: Consultas SQL incrustadas
* **Descripción:** Las consultas SQL están incrustadas directamente en los controladores y handlers.
* **Capa:** Datos.
* **¿Por qué usar este patrón?:** Un *Repository* encapsula el acceso a entidades y evita que la aplicación conozca SQL y las conexiones. No es *Unit of Work*: el *Repository* consulta y persiste; *Unit of Work* coordina atomicidad y confirmación (commit).
* **Cuándo NO aplica:** No aplicaría *Repository* a una consulta aislada de diagnóstico o a un script temporal que no forme parte de la aplicación principal.

### Problema 4: Falta de atomicidad en operaciones compuestas
* **Descripción:** El cobro a crédito y las inserciones de orden, refacciones y mecánico no forman una operación atómica.
* **Capa:** Datos / Aplicación.
* **¿Por qué usar este patrón?:** El problema es de transacción y consistencia: todas las escrituras deben confirmar o deshacerse juntas (*Unit of Work*). No es *Observer*: el *Observer* notifica reacciones, pero no garantiza el *rollback*.
* **Cuándo NO aplica:** No aplicaría *Unit of Work* a una operación de solo lectura o a una única escritura cuya atomicidad ya garantice la base de datos por defecto.

### Problema 5: Integración bancaria directa e interpretación XML manual
* **Descripción:** La integración bancaria se invoca directamente con `axios` y se interpreta el XML dentro del propio caso de uso.
* **Capa:** Integración.
* **¿Por qué usar este patrón?:** El banco tiene un contrato externo XML y la aplicación necesita una interfaz propia (ej. `PaymentGateway`). El *Adapter* traduce interfaces y formatos; *Facade* solo ofrece una interfaz simplificada sin que necesariamente exista incompatibilidad.
* **Cuándo NO aplica:** No aplicaría *Adapter* si la dependencia externa ya expone exactamente el contrato que usa el dominio y no se prevé sustituirla.

### Problema 6: Notificaciones acopladas a la transacción
* **Descripción:** El correo se envía directamente desde `CobroOrden`, antes de tener un commit confirmado en la base de datos.
* **Capa:** Aplicación / Integración.
* **¿Por qué usar este patrón?:** El caso de uso debería publicar un evento (`InscripcionPagada`) y un suscriptor (*Observer*) debería enviar el correo. No es *Unit of Work*: *Unit of Work* confirma datos; *Observer* reacciona a un hecho ya confirmado.
* **Cuándo NO aplica:** No aplicaría *Observer* si el resultado de la notificación debe decidir sincrónicamente si la operación completa tiene éxito y solo existe un consumidor permanente.

---

## Misión 2: Flujo de Ejecución de la Petición (Paso a Paso)

1. **Formulario HTML:** La vista de `/nueva` envía los campos a `POST /nueva`. Es la capa de presentación; no constituye por sí sola un MVC completo.
2. **`express.urlencoded`:** Middleware de Express que transforma el cuerpo `application/x-www-form-urlencoded` y lo expone en `req.body`.
3. **Middleware estático y de sesión global:** `express-session` recupera o crea `req.session`. Este es el mecanismo real de sesión, no el `AuthHandler` manual.
4. **Middleware de cookies escrito a mano:** Construye `req.cookies` desde la cabecera `Cookie`. Es un middleware de infraestructura, aunque resulta innecesario si se usa una biblioteca adecuada.
5. **Router de Express:** Encuentra `app.post('/nueva', ...)` y despacha la petición al callback registrado.
6. **`AuthHandler`:** Realiza autenticación y decide si continúa. En el código actual, está implementando una *Chain of Responsibility* manual, lo cual es incorrecto para este ecosistema.
7. **`BitacoraHandler`:** Registra `req.url` y delega al siguiente handler. También es una cadena manual; debería ser simplemente un middleware de Express colocado antes de la ruta.
8. **`CobroOrden`:** Es el supuesto servicio del caso de uso, pero actualmente mezcla responsabilidades: controlador, reglas de precio, integración bancaria, SQL, actualización de inventario, mecánico y correo. No hay una capa vacía adicional que aporte valor real.
9. **Cobro externo (si pago === 'tarjeta'):** `axios.post` llama al banco y el propio caso de uso interpreta la respuesta XML. Aquí es donde falta el *Adapter* de integración.
10. **Persistencia directa:** Se ejecutan `UPDATE` e `INSERT` mediante `conn.query`. No hay *Repository* ni *Unit of Work*. Las escrituras pueden quedar parcialmente aplicadas si ocurre un fallo en medio del proceso.
11. **Notificación:** Se crea un transporte de `Nodemailer` y se envía el correo directamente. No hay evento ni *Observer* real, y tampoco se verifica un commit exitoso antes del envío.
12. **Respuesta HTTP:** `CobroOrden` devuelve un fragmento de JavaScript al navegador, lo que acopla indebidamente el caso de uso puro con los objetos `req` y `res` de HTTP.

---

## Misión 3: Decisiones Arquitectónicas

### 1. ¿Cuándo corresponde usar un Front Controller?
Corresponde cuando todas o la mayoría de las peticiones necesitan un punto común para aplicar: autenticación, sesión, autorización, bitácora, manejo de errores, correlación o configuración global.
**En este proyecto:** Express ya actúa como *Dispatcher / Front Controller* del framework mediante la instancia `app` y sus rutas. No hace falta crear otra clase frontal. El error actual es colocar una cadena GoF (*Chain of Responsibility*) manual dentro de cada ruta.

### 2. Mecanismo que ya ofrece Express
Express ofrece herramientas nativas superiores:
* `app.use(...)` para establecer middleware global.
* `app.use('/ruta', middleware, router)` para middleware acotado por ruta.
* `express.Router()` para agrupar y segmentar rutas de manera lógica.
* Además, `express-session` ya ofrece la gestión de sesión completa.

### 3. Orden de la cadena (Middlewares) recomendado
1. ➔ **Autenticación (Primero):** Evita que un usuario no identificado ejecute operaciones protegidas y consuma recursos innecesarios.
2. ➔ **Cupo (Después):** Solo se evalúa la inscripción o el cupo para un usuario autenticado; puede delegar en una política o servicio de dominio.
3. ➔ **Bitácora (Después de las validaciones previas):** Registra el intento que efectivamente alcanzará el caso de uso (aunque podría existir una bitácora más amplia a nivel global para registrar rechazos tempranos).
4. ➔ **Caso de uso (Al final):** Ejecuta el cobro real, persiste en base de datos y publica el evento correspondiente posterior al commit.