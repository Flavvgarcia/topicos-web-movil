# Reporte de Análisis de Arquitectura

## Misión 1: Análisis de Problemas y Patrones

### 1. Mezcla de Código y Lógica
* **Problema observado:** HTML, CSS, JavaScript y consultas están mezclados en páginas como `index1.php:20-268`.
* **Capa:** Presentación.
* **Por qué este patrón:** *Page Controller* coordina una página concreta y *Template View* renderiza; no es “MVC” porque aquí no se está proponiendo una arquitectura completa de tres capas ni un modelo general.
* **Cuándo no aplicarla:** No aplicaría si fuera una página estática sin lógica ni datos dinámicos.

### 2. Vulnerabilidad en Gestión de Sesiones
* **Problema observado:** La sesión se acepta con cookies manipulables, como `admin_bypass`, en `index.php:4-5`.
* **Capa:** Políticas transversales.
* **Por qué esto no:** El *middleware* ejecuta autenticación en todas las peticiones y la *policy* decide permisos; no es solo *Front Controller*, porque entrar por un punto único no define si el usuario está autorizado.

### 3. Consultas Inseguras (Concatenación)
* **Problema observado:** Las consultas concatenan datos externos en `login1.php`, `Kardex1.php`, `pagar1.php`.
* **Capa:** Datos.
* **Por qué esto:** El *repositorio* centraliza la persistencia y el *mapper* transforma filas en objetos; no es *Active Record*, porque las entidades no deberían conocer ni construir SQL.
* **Esto no aplica porque:** No conviene introducir repositorios para una consulta aislada, inmutable y sin reutilización en un script pequeño.

### 4. Acoplamiento con Servicios Externos
* **Problema observado:** El XML del banco se construye y se interpreta directamente dentro del pago en `pagar1.php`.
* **Capa:** Integración.
* **Por qué esto:** El *adapter* convierte la interfaz interna de pago al protocolo XML externo; no es *Facade*, porque el problema principal no es simplificar varias operaciones, sino traducir contratos incompatibles.
* **Esto no aplica porque:** Si el proveedor ya expusiera exactamente la misma interfaz y formato que usa el dominio.

### 5. Falta de Atomicidad en Transacciones
* **Problema observado:** El pago modifica saldo, recibo e inscripciones mediante muchas consultas sin atomicidad en `pagar1.php`.
* **Capa:** Aplicación / Dominio.
* **Por qué este:** El *servicio* coordina el caso de uso y *Unit of Work* confirma o revierte el conjunto; no es *Observer*, porque observar eventos no garantiza que las escrituras sean atómicas.
* **Esto no aplica porque:** No usaría *Unit of Work* para una única lectura o una operación que no cambie varios agregados.

### 6. Inconsistencia de Modelos de Datos
* **Problema observado:** La documentación, el SQL y el código usan modelos incompatibles: `Estudiantes_V2`, `perfiles`, `usuarios_sys`, `kardex_historial` y tablas legacy.
* **Capa:** Datos / Integración.
* **Por qué esto:** La *capa anticorrupción* protege el modelo nuevo de nombres y reglas legacy; no es una *Facade*, porque no basta ocultar complejidad: hay que traducir conceptos y evitar contaminar el dominio.
* **Esto no aplica porque:** No aplicaría si se pudiera migrar todo a un único esquema canónico sin coexistencia temporal.

---

## Misión 2: Flujo de Ejecución Actual (Legacy)

1. **Navegador / Formulario de presentación:** 
   El usuario pulsa el botón de `pagar1.php:79-106`. El formulario envía el POST a `pagar.php`, no a `pagar1.php`.

2. **Endpoint HTTP directo:** 
   `pagar.php` recibe la petición, pero el archivo mostrado contiene variables basura y no presenta el flujo de pago completo. No existe un enrutador central ni un controlador claro.

3. **Procesamiento de aplicación:** 
   El flujo que aparentemente se pretendía ejecutar está dentro de `pagar1.php:5-72`. Ahí se leen directamente `$_POST` y se decide el método de pago. No hay un *Application Service*; el propio archivo mezcla controlador, reglas, SQL e integración.

4. **Integración con banco:** 
   Se construye XML y se llama a curl en `pagar1.php:15-25`. Esto debería ser `PaymentGatewayAdapter`, pero actualmente es código de infraestructura incrustado en la página.

5. **Persistencia del saldo:** 
   Se ejecuta directamente `UPDATE estado_cuenta` en `pagar1.php:28`. No hay *Repository*, validación de pertenencia del alumno ni comprobación del saldo.

6. **Persistencia del recibo:** 
   Se ejecuta directamente `INSERT INTO recibos` en `pagar1.php:29`. Tampoco se verifica si la operación ya había sido procesada.

7. **Persistencia de inscripciones:** 
   Se consulta `pre_alta`, se insertan materias y se eliminan registros en `pagar1.php:31-39`. Esto debería pasar por repositorios dentro de una misma *Unit of Work*, pero actualmente cada consulta queda independiente.

8. **Confirmación o rollback:** 
   No existe. Si falla el recibo o una inscripción después de descontar el saldo, el sistema queda en un estado parcial.

9. **Notificación:** 
   Se llama directamente a `mail()` en `pagar1.php:41-45`. Debería existir un `NotificationAdapter`, idealmente ejecutado después del commit mediante un evento/outbox.

10. **Respuesta HTTP:** 
    Se genera JavaScript con `alert()` y redirección desde el mismo archivo. Esto mezcla la respuesta de presentación con el caso de uso y no distingue correctamente errores técnicos, pagos rechazados y pagos pendientes.

---

## Misión 3: Políticas Transversales

Corresponde cuando varias peticiones necesitan pasar por las mismas operaciones iniciales:

* Iniciar sesión.
* Cargar configuración.
* Autenticar.
* Renovar o caducar sesión.
* Aplicar CSRF.
* Registrar la petición.
* Resolver la ruta.

**Diagnóstico actual:**
El proyecto tiene precisamente esa necesidad, pero no tiene un frontal único. Cada archivo PHP actúa como entrada independiente y puede saltarse controles críticos.