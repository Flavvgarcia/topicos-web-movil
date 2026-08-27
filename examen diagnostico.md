# Instituto Tecnológico de Morelia
## Examen Diagnóstico - Fundamentos de Ingeniería de Software
**Materia:** Tópicos Selectos de Tecnologías Web y Móvil  
**Profesor:** Jesús Eduardo Alcaraz Chávez  
**Nombre del alumno:** Flavio Garcia Herrera  
**Fecha:** 27/08/2026  

---

### Instrucciones
Responde de manera clara y concisa a cada una de las siguientes preguntas abiertas. El propósito de esta evaluación es medir tus conocimientos previos en Ingeniería de Software.

---

### 1. Metodologías
**¿Cuál es la diferencia principal entre una metodología de desarrollo tradicional (como Cascada) y una metodología ágil (como Scrum) frente a los cambios en los requisitos?**  
En Cascada todo está planeado desde el principio y si quieres cambiar algo a mitad de camino, es un dolor de cabeza porque es muy rígido. En cambio, con Scrum y las metodologías ágiles, vas trabajando por partes (*sprints*), lo que te permite adaptarte super bien y sin broncas si el cliente te sale con nuevos requerimientos o ajustes de última hora en medio del proyecto en blanco.

### 2. Requerimientos
**Explica la diferencia entre requerimientos funcionales y no funcionales, dando un ejemplo de cada uno aplicable a una plataforma web.**  
Los funcionales son lo que el sistema realmente hace o las acciones que permite; por ejemplo, que en una tienda online puedas armar tu carrito de compras y pagar. Los no funcionales, en cambio, se fijan más en el rendimiento o la calidad, como que la página cargue en menos de dos segundos aunque tenga un fondo blanco o un chorro de gente conectada al mismo tiempo.

### 3. Arquitectura
**Describe el modelo Cliente-Servidor y explica brevemente cómo se comunican el frontend y el backend en una aplicación web moderna.**  
Básicamente separa las cosas: el cliente es la interfaz que ves y usas en tu navegador o app, y el servidor es el que guarda la lógica pesada y la base de datos. Hoy en día, la comunicación se hace mandando peticiones HTTP desde el *frontend* hacia el *backend*, el cual procesa todo y te regresa la info lista y en blanco para que la pantalla la pueda mostrar sin broncas.

### 4. Bases de Datos
**¿En qué escenarios recomendarías utilizar una base de datos relacional (SQL) frente a una no relacional (NoSQL) para el almacenamiento de datos en una aplicación?**  
Una relacional te sirve perfecto cuando tus datos tienen que estar bien estructurados y relacionados entre sí, como en un sistema de banco o inventarios donde no puede haber errores. Una no relacional la usaría más bien si los datos cambian un montón de estructura o si necesito velocidad pura para leer y escribir, por ejemplo, en un chat en vivo o para guardar *logs* donde la base de datos arranca limpia y flexible.

### 5. APIs
**¿Qué es una API REST y qué papel fundamental juega en la integración entre una aplicación móvil y los servidores (backend)?**  
Es como un puente o traductor estándar que usa HTTP para que diferentes sistemas hablen entre sí con métodos como `GET` o `POST`.

### 6. Control de Versiones
**Explica la importancia de utilizar Git en un equipo de desarrollo de software y describe brevemente qué es un "merge conflict" (conflicto de fusión).**  
Te deja llevar el control de todo lo que cambias en el código, trabajar en equipo al mismo tiempo usando ramas y regresar a una versión estable si algo truena.  
*   **Merge conflict:** Pasa cuando dos personas tocan exactamente la misma línea de código en archivos distintos y Git se hace bolas sin saber cuál conservar, por lo que te toca a ti revisar y arreglarlo manualmente.

### 7. Pruebas
**¿Qué son las pruebas unitarias (unit testing) y por qué son cruciales para asegurar la calidad del software antes de su paso a producción?**  
Son pruebas automáticas que checan que lo más chiquito de tu código, como una sola función o método, jale exactamente como debe de ser de forma aislada.

### 8. POO (Programación Orientada a Objetos)
**Define los conceptos de encapsulamiento y polimorfismo, y menciona cómo ayudan a crear un código más mantenible.**  
El encapsulamiento oculta los datos internos de un objeto y solo deja ver lo necesario mediante métodos protegidos, y el polimorfismo permite que diferentes clases usen un método con el mismo nombre pero comportándose de manera distinta según convenga. Juntos hacen que el código sea mucho más ordenado, fácil de leer y de mantener a largo plazo.

### 9. Seguridad
**Explica la diferencia técnica entre "autenticación" y "autorización" en el contexto de seguridad de una aplicación.**  
La autenticación es básicamente comprobar quién eres y la autorización viene después y define qué puedes hacer o a qué pantallas puedes entrar según tu rol; o sea, te dice qué permisos tienes una vez adentro para que no andes viendo cosas que no te corresponden.