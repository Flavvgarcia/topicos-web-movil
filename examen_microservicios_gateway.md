           ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
             EXAMEN — MICROSERVICIOS, GATEWAY Y EL MONOLITO
                                CON API
              Tópicos Selectos de Tecnologías Web y Móvil

                            Eduardo Alcaraz
           ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━


Nombre:FLavio Garcia Herrera                       Calificación :_________


Indicaciones
════════════

  Este examen *no* se aprueba de memoria. Casi todas las preguntas
  tienen una respuesta que «suena bien» y está mal, o dos opciones
  defendibles donde se califica el argumento.

  Reglas:

  • Expliquen el *porqué*. Una conclusión sin razonamiento vale la mitad
    (o menos).
  • Cuando digan «depende», digan *de qué*.
  • No hace falta código que compile; sí hace falta que el diseño cierre
    (quién llama a quién, qué pasa si algo falla, qué ve el cliente).
  • Pueden dibujar. Un dibujo contradictorio con el texto resta.

  Puntaje: *100 puntos* (10 × 10).


1 — La frase del compañero (10 puntos)
══════════════════════════════════════

  En el pizarrón quedó:

        La API separa clientes del servidor; los microservicios
        separan servidores entre sí.

  Un compañero dice: «Entonces, si mi monolito Django expone una API y
  además lo meto en Docker, ya cumplí las dos mitades de la frase.»

  a) Desarmen su argumento. ¿Qué parte de la frase cree cumplir con la
  API? ¿Qué parte cree cumplir con Docker? ¿En qué se equivoca en cada
  una?

  Parte API: Cree cumplir la separación cliente-servidor, pero exponer endpoints solo da una interfaz de acceso; el código y los dominios siguen acoplados por dentro.
  Parte Docker: Cree cumplir la separación de servidores, pero Docker solo aísla un contenedor de software. Encapsular un bloque único de código solo empaqueta el monolito en otro formato.

  b) Den un contraejemplo mínimo: mismo código Django, *con* API y *con*
  Docker, que *sigamos* pudiendo llamar monolito. ¿Qué tendría que
  añadir (o partir) para que deje de serlo?

  Contraejemplo: Un Django con DRF empaquetado en una sola imagen de Docker conectada a una única base de datos PostgreSQL.
  Para dejar de serlo: Romper la base de código en repositorios independientes con procesos y bases de datos separadas por cada dominio (ej. Usuarios y Pedidos).

  c) ¿La frase del pizarrón habla de *tecnologías* (Docker, REST) o de
  *límites de proceso / despliegue*? Defiendan su lectura con lo que
  vieron en clase.

  Habla de límites de proceso y despliegue. Las tecnologías (REST, Docker) son simples herramientas de implementación; la arquitectura define dónde terminan la memoria, los datos y la ejecución de cada servicio para evitar fallas en cascada.


2 — Tres rutas, una trampa (10 puntos)
══════════════════════════════════════

  El equipo muestra este `urls.py' de un solo proyecto Django:

  ┌────
  │ path("usuarios/", ...),
  │ path("pedidos/", ...),
  │ path("recomendar/", ...),
  └────

  y este `docker-compose.yml':

  ┌────
  │ services:
  │   api:
  │     build: .
  │     ports: ["8000:8000"]
  │     deploy:
  │       replicas: 3
  └────

  Afirman: «Tres rutas + tres réplicas = microservicios con alta
  disponibilidad.»

  a) Separen las dos ideas que mezclaron (rutas vs réplicas). ¿Qué
  problema resuelve cada una y cuál *no* resuelve?

  Rutas: Organizan internamente los endpoints del código (no resuelven escalabilidad ni desacoplamiento).
  Réplicas: Dan alta disponibilidad clonando instancias (no crean microservicios; solo duplican el mismo monolito).

  b) Si una de las tres réplicas se cae a mitad de un `POST /pedidos',
  ¿el sistema dejó de ser monolito? ¿Por qué sí o por qué no?

  No,La caída de un nodo afecta la infraestructura, no la arquitectura. Mover un sistema acoplado a 3 contenedores solo da un monolito redundante.

  c) Rediseñen *solo con palabras* el `compose' para que sí ilustre
  microservicios (nombres de servicios y responsabilidades; no hace
  falta YAML perfecto). ¿Cuántas *imágenes distintas* mínimo y por qué
  esa cantidad?

  Mínimo 3 imágenes distintas: `usuarios-service`, `pedidos-service` y `recomendar-service`. Se requiere esa cantidad porque cada contenedor debe encapsular únicamente el código y contexto de su propio dominio.


3 — El gateway que «hace de más» (10 puntos)
════════════════════════════════════════════

  Les proponen este diseño «para no complicar»:

        Un solo contenedor nginx+Python que: valida el JWT, elige
        el medio con el `.joblib', guarda el pedido en SQLite,
        manda el correo y devuelve el HTML.

  Lo llaman «API Gateway».

  a) ¿Es un gateway? Si no, ¿qué es? Usen la analogía del recepcionista
  (o otra) y digan qué oficios mezclaron.

  No es un gateway, es un monolito saturado. En la analogía: el recepcionista (gateway) debe revisar identidades y pasar visitas; aquí además cocina, atiende el teléfono, arregla equipos y reparte paquetes.

  b) Mencionen *tres* consecuencias malas de ese diseño cuando el modelo
  pesado se reentrena cada semana y el correo empieza a fallar.

  1. Reentrenar el `.joblib` satura el CPU y congela tareas simples como validar un JWT.
  2. Un fallo en la librería de correo puede tirar el proceso entero y frenar las ventas.
  3. Escalar el punto de entrada obliga a duplicar el modelo pesado en memoria RAM en cada nodo.

  c) Partan ese contenedor en piezas. Para cada pieza digan: ¿queda en
  el borde (gateway) o detrás? No inventen diez servicios: el mínimo que
  separe borde, logística e (si hace falta) aviso.

  • API Gateway (Borde): Valida JWT y rutea tráfico.
  • Servicio Pedidos (Detrás):Guarda transacciones en la BD.
  • Servicio Recomendación (Detrás):Ejecuta el `.joblib`.
  • Worker de Correos (Detrás):Procesa envíos de manera asíncrona.


4 — `localhost', la trampa favorita (10 puntos)
═══════════════════════════════════════════════

  Pedidos corre en un contenedor. Recomendar en otro, misma Compose.
  Alguien «arregla» la conexión así:

  ┌────
  │ RECOMENDAR_URL=http://127.0.0.1:8002
  └────

  En su laptop, *sin* Docker, ese `curl' a `127.0.0.1:8002' sí
  funcionaba.

  a) Expliquen por qué en Compose ese arreglo suele fallar *aunque*
  Recomendar esté sano. ¿A qué proceso apunta `127.0.0.1' desde dentro
  de Pedidos?

  Dentro de un contenedor, `127.0.0.1` apunta a su propio loopback local. Pedidos busca el puerto 8002 en sí mismo y falla porque Recomendar corre en otro contenedor.

  b) ¿Por qué `http://recomendar:8002' sí tiene chance de funcionar?
  ¿Quién «inventa» el nombre `recomendar'?

  Funciona porque el DNS interno de Docker traduce el nombre del servicio a la IP privada del contenedor. El nombre lo "inventa" Docker Compose a partir de la llave definida en el `docker-compose.yml`.

  c) Ahora Recomendar se muda a la laptop de otro alumno
  (`192.168.0.57'). Escriban la URL que pondrían *y* dos cosas que
  tienen que ser verdad en la WiFi para que no sea teatro (puertos,
  firewall, etc.). Si solo cambian la URL y nada más, ¿qué puede seguir
  fallando?

  URL: `http://192.168.0.57:8002`
  WiFi: Firewall remoto abriendo puerto 8002 y red sin aislamiento de clientes (AP Isolation).
  Sigue fallando: Si la app de Carla escucha solo en `127.0.0.1` en vez de `0.0.0.0`.


5 — Fallo parcial con números (10 puntos)
═════════════════════════════════════════

  Escenario medido en laboratorio (inventado pero coherente):

  • 1000 `POST /pedidos' por hora en promedio.
  • Recomendar tarda 200 ms cuando está bien.
  • El 5 % del tiempo Recomendar no responde (timeout 3 s).
  • Pedidos, si hay timeout, usa plan B (moto) y responde 200 en 50 ms.
  • El gateway añade 5 ms.

  a) En la hora «normal» (Recomendar bien), ¿el cliente espera ~200 ms o
  ~3 s? ¿Quién pone el tiempo dominante?

  Espera ~205 ms ($200\text{ ms} + 5\text{ ms}$). El tiempo dominante lo marca el servicio Recomendar.

  b) En el 5 % malo, con plan B: ¿el cliente ve error o pedido creado?
  ¿Qué «mentira útil» están aceptando (medio quizá no óptimo)?

  Ve pedido creado (200 OK). La mentira útil es asignar un transporte por defecto (moto) sacrificando la sugerencia óptima con tal de asegurar la venta.

  c) Un compañero dice: «Mejor que Pedidos *reintente* 10 veces hasta
  que Recomendar vuelva; así nunca usamos plan B.» ¿Qué le pasa a la
  cola de peticiones y al gateway si Recomendar está caído 10 minutos?
  ¿Plan B siempre es peor que reintentar? Argumenten.

  La cola de peticiones y los hilos del gateway colapsarían acumulando timeouts de $3\text{ s}$, tirando todo el sistema. El plan B es superior porque permite degradación elegante sin colapsar el negocio.


6 — Apagan el «edificio equivocado» (10 puntos)
═══════════════════════════════════════════════

  Tres contenedores: gateway, pedidos, recomendar. Hacen tres
  experimentos *por separado*:

  1. `docker compose stop gateway'
  2. `docker compose stop pedidos'
  3. `docker compose stop recomendar'

  Para *cada* experimento digan qué ve el navegador en `http://IP:8080/'
  (página / crear pedido) y qué sigue vivo por dentro. No basta «no
  jala»: digan *qué* no jala y qué sí podría seguir respondiendo si
  alguien hablara por dentro de la red.

  Luego: ¿en cuál de los tres el sistema «parece» más muerto hacia
  afuera aunque más piezas sigan arriba? ¿Qué les enseña eso del borde?

  1. `stop gateway`: Navegador muestra `ERR_CONNECTION_REFUSED`. Pedidos y Recomendar siguen ejecutándose aislados en la red interna.
  2. `stop pedidos`: Navegador devuelve error 502/504 al crear pedido. Gateway y Recomendar siguen vivos.
  3. `stop recomendar`: Navegador recibe 200 OK (activa plan B). Gateway y Pedidos están vivos.

  Conclusión: Parece más muerto en el Experimento 1. Enseña que el borde es la cara pública del sistema: si este cae, la salud interna de los servicios se vuelve invisible para el usuario.


7 — WiFi del salón, conflicto de roles (10 puntos)
══════════════════════════════════════════════════

  Cuatro alumnos:

  • Ana: gateway + HTML
  • Beto: pedidos
  • Carla: recomendar
  • Diana: «yo también levanté un gateway en mi laptop por si Ana falla»

  Todos en la misma WiFi. El cliente (otro equipo) configura a veces la
  IP de Ana y a veces la de Diana.

  a) ¿Qué problema de *diseño* introdujo Diana aunque «todo responda
  200»? (Piensen en una sola puerta vs dos puertas.)

   Eliminó la puerta única de entrada, fragmentando métricas, ruteo y control de seguridad al tener dos puntos de acceso no sincronizados.

  b) Beto configuró `RECOMENDAR_URL=http://localhost:8002' porque en su
  máquina desarrolló Pedidos y Recomendar juntos. En el salón Recomendar
  está en la laptop de Carla. ¿Qué falla y cómo lo corrigen *sin* juntar
  otra vez los dos servicios en la laptop de Beto?

   Falla: Pedidos busca el servicio localmente.
   Corrección: Cambiar la variable a `RECOMENDAR_URL=http://<IP_Carla>:8002` y asegurar que Carla escuche en `0.0.0.0`.

  c) La WiFi del campus aísla clientes («AP isolation»). `ping' entre
  alumnos falla. ¿Pueden hacer la práctica tal cual? Si no, propongan
  *un* plan B de laboratorio que preserve la lección «procesos
  separados» (aunque no sea WiFi del salón).

  No se puede en LAN. Plan B: Usar túneles HTTP (`ngrok`) para exponer los puertos locales a URLs públicas y comunicarse a través de Internet.


8 — El front que conoce demasiado (10 puntos)
═════════════════════════════════════════════

  El HTML hace esto al enviar el formulario:

  ┌────
  │ await fetch("http://192.168.1.20:8002/sugerir", { ... }); // Recomendar
  │ await fetch("http://192.168.1.30:8001/pedidos", { ... }); // Pedidos
  └────

  El autor dice: «Más rápido: me salto el gateway.»

  a) Listen *tres* problemas (seguridad, cambio de IP, orden de
  llamadas, consistencia del folio, CORS, etc.).

  1. Exposición directa de la red e IPs privadas al cliente.
  2. Errores de CORS al llamar diferentes orígenes desde el navegador.
  3. Acoplamiento de la lógica y orden de llamadas dentro del frontend.

  b) Rediseñen la secuencia correcta con gateway: *una* llamada desde el
  navegador. ¿Quién habla con Recomendar y por qué no el navegador?

  Secuencia: Navegador hace `POST /pedidos` al Gateway. El servicio Pedidos recibe la orden y llama internamente a Recomendar. El navegador no debe orquestar la lógica privada del backend.

  c) El selector de modelo (`arbol=|=bosque=|=red') va en el JSON. Si el
  front llama directo a Recomendar *y* a Pedidos, ¿quién debería crear
  el folio para no duplicar lógica? ¿Qué invariante rompen si los dos
  «crean» algo?

  Debe crearlo Pedidos. Si los dos crean algo, rompen la invariante de **identidad única del recurso, generando registros duplicados o huérfanos.


9 — El LLM vendedor (10 puntos)
═══════════════════════════════

  Producto: tienda + envíos.

  • Modelo `.joblib': medio de transporte.
  • LLM: texto para convencer al cliente en la ficha del producto.

  Un arquitecto propone *un solo* microservicio `cerebro' que: recibe el
  carrito, elige el medio con el árbol *y* genera el pitch de ventas con
  el LLM, todo en la misma petición `/inteligente'.

  a) Critiquen el nombre y el corte. ¿Qué dos oficios distintos
  mezclaron? ¿Qué falla cuando el LLM tarda 8 s y el despacho necesita
  responder en < 500 ms?

  Mezclaron **logística operacional** con **marketing**. Si el LLM tarda 8s, viola el requisito de respuesta del despacho (<500ms) y congela la operación de compras.

  b) Propongan dos endpoints (o dos servicios) y digan cuál usa el
  *front de tienda* y cuál el *flujo de pedido/envío*.

  • `POST /api/logistica/transporte` -> Usado por el flujo de **pedido/envío**.
  • `GET /api/marketing/pitch` -> Usado por el **front de tienda**.

  c) El LLM vive en un server con GPU en otra red. El front de tienda
  ¿apunta a la GPU o al gateway? Si mañana cambian de proveedor LLM
  (otro host), ¿qué archivo/config quieren tocar *una sola vez*?

  Apunta al gateway. Si cambia la infraestructura, solo se modifica la ruta proxy en el archivo de configuración del Gateway.


10 — El inversionista y la semana imposible (10 puntos)
═══════════════════════════════════════════════════════

  Hoy:

  • Panel interno: monolito Django (funciona, poco tráfico).
  • App clientes: gateway + pedidos + recomendar.
  • Idea: LLM de ventas (aún no).
  • Equipo: 5 personas.
  • Inversionista: «Todo a microservicios en 7 días, incluido el panel,
    o no hay dinero.»

  a) Escriban la respuesta al inversionista (máx. 8 líneas) con postura
  *y* riesgo concreto si obedecen a ciegas.

  Migrar el panel interno a microservicios en 7 días no agrega valor y genera alto riesgo de fallas operativas por forzar cambios en un sistema estable. Priorizaremos aislar y estabilizar los servicios orientados a clientes, que son la fuente de ingresos. Dividir código sin necesidad operativa solo gastará horas de desarrollo críticas.

  b) Ordenen *estas cuatro* iniciativas de más a menos urgente para
  *esta* semana y justifiquen el orden (no el catálogo soñado):

  1. Partir el panel Django en 6 microservicios.
  2. Sacar el reentrenamiento del `.joblib' a un pipeline que no tumbe
     Pedidos.
  3. Meter auth + rate limit en el gateway público.
  4. Subir el LLM de ventas a producción sin timeouts ni plan B.

  Orden: `3 -> 2 -> 4 -> 1`
  • **3:** Proteger el borde (seguridad básica).
  • **2:** Evitar que el entrenamiento del modelo tire el flujo de compras.
  • **4:** Aporta ingresos inmediatos al negocio.
  • **1:** Desperdicio de tiempo en un panel interno de bajo tráfico.

  c) Inventen *una* métrica que les diga «ya toca partir X del monolito»
  (no «porque está de moda»). Tiene que ser observable (tiempo, errores,
  despliegues, equipo…).

  **Frecuencia de fallas en despliegues cruzados:** Ocurrencia de 2 o más desacuerdos/rollbacks semanales causados porque un equipo tocó el código de otro dominio en el repositorio compartido.

  d) Cierren con una frase propia (no copien el apunte) que capture:
  ¿microservicios son el objetivo o la herramienta?

  *Los microservicios son la herramienta para aislar fallas y escalar equipos, no un trofeo de arquitectura.*