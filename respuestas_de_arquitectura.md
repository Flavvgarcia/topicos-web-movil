# Respuestas: Arquitectura de la Plataforma TWAM

**1. Clasificar la plataforma actual**
Es un **monolito modular con API**. La evidencia es que el frontend le pega por red a endpoints como `/entregas/recomendar/` (esa es la API), pero del lado del servidor, todos los módulos de lógica e IA corren acoplados en un mismo proceso de Django compartiendo la misma memoria.

**1.1 Clasificar la plataforma actual**

```text
[ Cliente (App Flutter/Web) ]
          |
          | Petición HTTP (API) -> /entregas/recomendar/
          v
+---------------------------------------------------+
|            Servidor Backend (Django)              |
|             (Un solo proceso en memoria)          |
|                                                   |
|  [ recomendador.py ] <--> [ reglas.py ]           |
|       ( + .joblib )             |                 |
|                                 v                 |
|                           [ medios.py ]           |
+---------------------------------------------------+

**2. La trampa de los endpoints**
Tener distintas rutas HTML o endpoints solo significa que tenemos una API que separa al cliente del servidor. Para que realmente sean microservicios, los módulos tendrían que correr en servidores o contenedores distintos hablándose por red, no solo estar en archivos separados.

**3. La misma operación en dos mundos**
*   **En el monolito actual:** La vista simplemente hace un `import` del recomendador en la memoria RAM. Si sugiere "dron", el módulo local `reglas.py` entra al quite y lo cambia a "moto" ahí mismo, en milisegundos.
*   **Como microservicio:** La vista tendría que hacer una petición HTTP (un salto de red) al servidor de IA. Si la red falla o el servicio se cae, el backend de pedidos no se puede quedar colgado: ignoraría a la IA y usaría sus reglas locales como plan B para sacar la entrega.

**4. Decisión de la empresa**
Lo mejor es **extraer únicamente el servicio de IA** y dejar el resto como monolito. 
1. Nos permite actualizar y reiniciar el modelo cada semana sin tirar el sistema de registro de pedidos. 
2. Como el Random Forest pesa 50MB, podemos asignarle más RAM solo a ese servicio sin encarecer los demás servidores. 
3. El código de pedidos es súper estable; partirlo en más microservicios solo metería latencia y complejidad de a gratis.

**5. Un solo corte**
*   **Se va al microservicio:** Solo `recomendador.py` y los pesos del modelo (`.joblib`).
*   **Se queda en el backend:** Las vistas, `medios.py` y `reglas.py`.
*   **Por qué este corte:** Sacamos la parte pesada e inestable (la IA) a su propio espacio, pero mantenemos todas las reglas críticas del negocio en la memoria principal. Si hacemos "un servicio por archivo", el sistema se volvería lentísimo por estar mandando peticiones de red para cada pequeña validación.