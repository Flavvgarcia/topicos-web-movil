# Análisis de Arquitectura: PasoFit Flutter

---

## Misión 1: Análisis de Problemas y Patrones

### Problema 1: Notificaciones acopladas mediante listas globales
* **Descripción:** `CronometroPantalla` y `PesoPantalla` se registran en listas globales de callbacks.
* **Capa:** Aplicación / Dominio.
* **¿Por qué usar este patrón?:** Hay un sujeto que publica cambios y varios observadores que reaccionan (*Observer*). No es *Mediator*, porque el bus no coordina una conversación entre objetos ni contiene reglas de interacción.
* **Cuándo NO aplica:** No aplicaría si solo existe un consumidor y basta con devolver el valor directamente.

### Problema 2: Cálculo de calorías rígido
* **Descripción:** `RutinaPantalla` contiene una cadena de `if` para calcular las calorías según el tipo.
* **Capa:** Aplicación / Dominio.
* **¿Por qué usar este patrón?:** El cálculo cambia según una política seleccionable (*Strategy*). No es *Factory*: *Factory* crea objetos, mientras *Strategy* encapsula algoritmos intercambiables.
* **Cuándo NO aplica:** No aplicaría para dos casos triviales, estables y sin probabilidad de crecimiento.

### Problema 3: Acceso a datos directo sin transaccionalidad
* **Descripción:** La pantalla abre SQLite y ejecuta dos `INSERT` directamente.
* **Capa:** Datos / Aplicación.
* **¿Por qué usar este patrón?:** El *Repository* debe ocultar SQL y ofrecer operaciones del dominio; *Unit of Work* debe hacer atómicas las escrituras relacionadas. No es solo *DAO*, porque un DAO expone una abstracción centrada en tablas, no una operación del negocio.
* **Cuándo NO aplica:** *Unit of Work* no hace falta si solo existe una escritura independiente y no hay consistencia entre varias operaciones.

### Problema 4: Integración HTTP directa
* **Descripción:** La pantalla de rutina llama directamente a `/fit/mail`.
* **Capa:** Integración.
* **¿Por qué usar este patrón?:** El *Adapter* traduce el contrato HTTP externo a una interfaz local, aislando al dominio de HTTP. No es *Facade*: *Facade* simplifica varias operaciones; *Adapter* adapta interfaces incompatibles.
* **Cuándo NO aplica:** No aplicaría en un prototipo de una sola llamada, sin contrato externo ni posibilidad de sustitución.

### Problema 5: Orquestación fragmentada
* **Descripción:** `InicioPantalla` realiza ocho peticiones y concatena sus respuestas.
* **Capa:** Integración / Aplicación.
* **¿Por qué usar este patrón?:** Una Fachada (*Facade*) ofrece una entrada única para coordinar varios módulos y devolver un resultado coherente. No es *Adapter*, porque aquí no se traduce una interfaz: se agrupan varias operaciones.
* **Cuándo NO aplica:** No aplicaría si solo hubiera una fuente o si el cliente necesitara controlar individualmente cada petición.

### Problema 6: Políticas repetidas en endpoints
* **Descripción:** Autenticación, caducidad de sesión y bitácora se repetirían en cada endpoint.
* **Capa:** Políticas transversales.
* **¿Por qué usar este patrón?:** Un punto frontal recibe las peticiones y una cadena aplica políticas comunes antes del caso de uso. No es “MVC” ni “hexagonal”: esos nombres describen una organización general, no el mecanismo concreto de interceptación.
* **Cuándo NO aplica:** No aplicaría a una operación local sin peticiones HTTP ni políticas compartidas.

---

## Misión 2: Flujo de Ejecución de la Petición (Paso a Paso)

1. **Clic del usuario o `POST /inscripciones/{id}/pago`:** El navegador o cliente HTTP inicia la petición. Todavía no hay un patrón de negocio.
2. **Router:** Asocia el método y la URL con el controlador de pago. Es enrutamiento, no *Facade* ni *Controller* del dominio.
3. **Front Controller:** Recibe todas las peticiones HTTP o todas las de la aplicación y establece el contexto común: sesión, usuario, correlación y manejo de errores.
4. **Middleware o Filter de autenticación:** Valida la sesión o token antes de permitir que el caso de uso continúe.
5. **Middleware o Filter de cupo:** Aplica un límite técnico transversal, por ejemplo, cantidad máxima de inscripciones simultáneas o por usuario.
6. **Middleware o Filter de bitácora:** Registra quién intentó ejecutar la operación, qué recurso solicitó y el resultado. Puede implementarse como middleware “around” para registrar también éxito o fallo.
7. **Controlador HTTP:** Valida el formato, convierte el cuerpo HTTP en un comando o DTO y llama al servicio de aplicación. No debe calcular precios, modificar SQL ni llamar directamente al proveedor de pagos.
8. **Servicio de aplicación (`PagarInscripcion`):** Coordina el caso de uso: carga la inscripción, verifica su estado, solicita el importe y coordina persistencia y pago. No es una capa vacía: contiene la secuencia de la operación.
9. **Política o Strategy de cobro:** Selecciona la regla de cálculo o modalidad de pago cuando existen varias alternativas. Si solo hay una regla fija, no se necesita *Strategy*.
10. **Gateway de pago (Adapter):** Convierte el contrato interno en la solicitud que entiende Stripe, PayPal u otro proveedor. Aísla las diferencias externas y devuelve un resultado interno.
11. **Repositories dentro de una Unit of Work:** Persisten el pago y el nuevo estado de la inscripción. La *Unit of Work* confirma o revierte juntas las modificaciones que deban ser atómicas.
12. **Publicación o envío de notificación:** Tras confirmar el pago, se publica un evento o se llama a un servicio de notificaciones. Si la notificación debe ser resistente a fallos, conviene una cola y el patrón *Outbox*; no se debe enviar un correo antes de confirmar la transacción.
13. **Respuesta del controlador:** Devuelve `200`, `201` o un error apropiado. El router y el *Front Controller* completan la respuesta HTTP.

---

## Misión 3: Decisiones Arquitectónicas

### 1. ¿Cuándo corresponde usar un Front Controller?
Corresponde cuando todas o muchas peticiones necesitan políticas comunes: autenticación, sesión, trazabilidad, manejo uniforme de errores, límites de acceso o métricas. Centraliza ese punto de entrada y evita que cada controlador repita la misma secuencia.
*Nota sobre Flutter:* No corresponde crear un Front Controller dentro de cada pantalla Flutter. La sesión y la bitácora deben resolverse en el backend HTTP.

### 2. Orden de cadena recomendado
El flujo debe seguir este orden:
1. **Front Controller** (Punto de entrada)
2. ➔ **Autenticación**
3. ➔ **Cupo de inscripciones**
4. ➔ **Bitácora**
5. ➔ **Controlador**
6. ➔ **Servicio del caso de uso**