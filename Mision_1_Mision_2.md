### Misión 1 — El portal no es un patrón

El título nos dice que no podemos solucionar todo el portal diciendo "usamos MVC"; cada conflicto tiene su propia capa y estructura.

* **Validaciones (Intercepting Filter):** El problema no es el portal entero, es la entrada. En vez de pedir credencial salón por salón, ponemos un guardia (filtro) en la puerta principal.
* **Consultas de BD (Repository):** El problema está aislado en los datos. En vez de que cada vista busque en los archivos, asignamos un bibliotecario exclusivo para entregarte la información limpia.
* **App vs Kiosco (BFF):** El problema es la presentación. El sistema central no cambia, solo creamos un "traductor a la medida" para que la app y el kiosco reciban exactamente lo que necesitan.
* **Código bancario (Strategy + Adapter):** El problema es la integración. Usamos un "enchufe adaptador" para que el banco hable el mismo idioma que el reglamento de la universidad.
* **Correos y cobros dobles (Unit of Work + Observer):** El problema es el dominio. Primero bloqueamos tu expediente para no cobrarte doble, y luego un "mensajero" envía tu correo en segundo plano para no detener la inscripción principal.
* **Servicios caídos (Circuit Breaker):** El problema es la red. Si el validador de CURP se cae, "colgamos la llamada" rápido para no congelar todo el portal.

### Misión 2 — Una petición, varios patrones

El título exige demostrar cómo un solo movimiento (dar clic en pagar o enviar el `POST`) desata una reacción en cadena ordenada a través de distintas herramientas.

1. **La Recepcionista (Front Controller):** Recibe tu `POST` inicial y lo canaliza.
2. **El Guardia (Intercepting Filter):** Revisa tu sesión antes de dejarte pasar.
3. **El Capturista (Application Controller):** Lee los datos de tu formulario (cuánto y cómo pagas).
4. **El Supervisor (Unit of Work):** Aplica un candado para asegurar que no haya cobros dobles.
5. **El Cajero Externo (Strategy + Adapter):** Ejecuta el cobro con el banco y traduce la respuesta a "acreditado".
6. **El Archivista (Repository):** Guarda el estado de tu pago en la base de datos de forma segura.
7. **El Mensajero (Observer):** Al final, notifica a Control Escolar y dispara tu recibo por correo.