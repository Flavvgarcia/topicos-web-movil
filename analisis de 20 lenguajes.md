# Análisis de Patrones de Diseño en 20 Lenguajes de Programación

Los patrones de diseño tradicionales (como los de la *Gang of Four* o GoF) fueron concebidos principalmente para resolver deficiencias o estructurar el código en lenguajes orientados a objetos con tipado estático (como C++ o Java). Al cambiar de paradigma, muchos de estos patrones se vuelven innecesarios, redundantes o incluso antipatrones.

## 1. 20 Lenguajes, Paradigmas y su Relación con los Patrones

| Lenguaje | Paradigma Principal | Relación con los Patrones de Diseño (GoF) |
| :--- | :--- | :--- |
| **1. Java** | Orientado a Objetos (OOP) | Implementación clásica de GoF (Factory, Singleton, Observer). |
| **2. C#** | OOP / Multiparadigma | Clásica, pero simplificada por delegados (reemplaza Observer/Command). |
| **3. C++** | Multiparadigma / OOP | Clásica. Usa RAII para el manejo de recursos (Singleton es riesgoso). |
| **4. Python** | OOP / Dinámico | Muchos patrones son nativos o innecesarios por el tipado dinámico (*duck typing*). |
| **5. JavaScript** | Multiparadigma / Prototipado | Patrones basados en prototipos, módulos y funciones de primera clase. |
| **6. TypeScript** | OOP / Estructural | Vuelve a traer patrones clásicos de Java/C# al ecosistema web. |
| **7. Ruby** | OOP Puro / Dinámico | Metaprogramación hace que patrones como Builder o Proxy sean casi invisibles. |
| **8. Go (Golang)** | Concurrente / Estructural | **Rompe patrones clásicos.** Sin herencia, prefiere composición e interfaces implícitas. |
| **9. Rust** | Multiparadigma / Sistemas | **Rompe patrones clásicos.** El *Borrow Checker* dificulta el estado mutable compartido. |
| **10. Haskell** | Funcional Puro | **Rompe patrones clásicos.** No hay estado; usa Mónadas y funciones de orden superior. |
| **11. Clojure** | Funcional / Lisp | **Rompe patrones clásicos.** Inmutabilidad y macros reemplazan la estructura OOP. |
| **12. Scala** | OOP / Funcional | Combina GoF con programación funcional (Pattern Matching, Traits). |
| **13. Kotlin** | OOP / Funcional | Clásica, pero elimina el *boilerplate* (Singletons son nativos con `object`). |
| **14. Swift** | Orientado a Protocolos | Patrones basados en protocolos (interfaces) y extensiones en lugar de herencia. |
| **15. PHP** | OOP / Procedural | Uso clásico de MVC, Dependency Injection y Factory en frameworks (Laravel). |
| **16. Elixir** | Funcional / Concurrente | **Rompe patrones clásicos.** Basado en el modelo de Actores, no en objetos. |
| **17. Erlang** | Funcional / Concurrente | Igual que Elixir; usa OTP (Open Telecom Platform) en lugar de GoF. |
| **18. Lua** | Multiparadigma / Scripts | Tablas y metatablas actúan como un sistema de patrones flexible (prototipado). |
| **19. Common Lisp** | Multiparadigma / Lisp | El sistema de macros permite crear tus propios patrones a nivel sintáctico. |
| **20. Julia** | Multiple Dispatch | **Rompe patrones clásicos.** El despacho múltiple hace obsoletos patrones como Visitor. |

---

## 2. Análisis de Lenguajes que "Rompen" los Patrones de Diseño

Cuando se intentan forzar los patrones tradicionales (OOP) en los siguientes lenguajes, se genera código complejo, ineficiente o que va en contra de la filosofía idiomática del lenguaje. A continuación, se desglosa el problema estructural por lenguaje:

### Rust
Rust utiliza un sistema de propiedad (*Ownership*) y préstamo (*Borrowing*) para garantizar la seguridad de la memoria sin un recolector de basura.
*   **El Problema:** Patrones que requieren estado mutable compartido (como el **Observer**, **State** o estructuras de datos con referencias circulares) son extremadamente difíciles de implementar. El compilador bloquea las múltiples referencias mutables al mismo objeto por diseño de seguridad.
*   **La Solución / Ruptura:** Rust prefiere arquitecturas basadas en mensajes (canales) o el uso explícito de celdas de interior mutable (`Rc<RefCell<T>>`), lo cual es deliberadamente verboso para desincentivar su uso. Patrones como **Visitor** se reemplazan de manera nativa y más elegante con *Pattern Matching* y estructuras `enum`.

### Haskell (y lenguajes funcionales puros como Clojure)
Haskell no tiene clases, objetos mutables, ni herencia.
*   **El Problema:** La gran mayoría de los patrones GoF existen para manejar el estado interno de los objetos o encapsular comportamientos polimórficos mutables. En Haskell, el estado mutable no existe por defecto.
*   **La Solución / Ruptura:** 
    *   El patrón **Strategy** es redundante; simplemente se pasa una función como argumento (Funciones de Orden Superior).
    *   El patrón **Command** se vuelve irrelevante, ya que cualquier función puede ser evaluada de forma perezosa (*lazy evaluation*).
    *   El patrón **Singleton** no tiene sentido sin estado mutable.
    *   En lugar de la estructura GoF, Haskell utiliza conceptos matemáticos teóricos como Mónadas, Functores y Aplicativos para lidiar con efectos secundarios y transformaciones.

### Go (Golang)
Go fue diseñado de manera intencional para ser simple, careciendo de herencia basada en clases (no existe `extends`).
*   **El Problema:** Patrones estructurales que dependen fuertemente de jerarquías de clases y herencia (como **Template Method** o el **Factory Method** clásico) no se pueden implementar de manera directa. 
*   **La Solución / Ruptura:** Go rompe la necesidad de herencia imponiendo el uso de composición (incrustar estructuras dentro de otras) y tipado estructural (interfaces implícitas). Para el patrón **Observer**, Go promueve el uso de su concurrencia nativa (Goroutines y Channels) bajo su filosofía: *"No te comuniques compartiendo memoria; comparte memoria comunicándote"*.

### JavaScript (y lenguajes de tipado dinámico)
Aunque modernamente tiene sintaxis de `class`, JavaScript utiliza herencia prototípica y tipado dinámico (*Duck Typing*).
*   **El Problema:** Implementar interfaces estrictas (requeridas por patrones como **Adapter**, **Bridge** o **Proxy**) requiere escribir excesivo código manual de validación de tipos, lo cual destruye la fluidez natural del lenguaje.
*   **La Solución / Ruptura:** 
    *   El patrón **Singleton** en JS es innecesariamente complejo en su forma clásica; se logra simplemente exportando un objeto literal (`const mySingleton = {}`).
    *   El patrón **Module** (nativo desde ES6) cubre de fábrica la necesidad de la mayoría de los patrones de encapsulamiento estructural.

### Julia
Julia es un lenguaje enfocado en cómputo científico que utiliza el *Multiple Dispatch* (Despacho Múltiple) como su pilar central.
*   **El Problema:** En la programación orientada a objetos clásica (Single Dispatch), el método que se ejecuta depende únicamente del objeto (clase) que lo llama. Esto obliga a crear constructos pesados como el **Visitor** para agregar operaciones a una jerarquía de clases sin modificarla internamente.
*   **La Solución / Ruptura:** En Julia, el despacho múltiple elige el método basándose en los tipos de *todos* los argumentos involucrados al mismo tiempo. Esto hace que el patrón **Visitor** quede obsoleto, ya que se puede definir un comportamiento nuevo para cualquier combinación de tipos directamente desde fuera, sin tocar las estructuras originales.

### Elixir y Erlang
Lenguajes orientados a la alta concurrencia y tolerancia a fallos.
*   **El Problema:** El patrón **Observer** clásico asume que el sujeto y los observadores viven en el mismo hilo/proceso y comparten memoria de alguna manera.
*   **La Solución / Ruptura:** Utilizan el modelo de Actores (mediante OTP). Cada "objeto" es un proceso ligero e independiente que no comparte estado con los demás. Si se requiere un comportamiento de observación, los procesos simplemente se envían y reciben mensajes, haciendo obsoleto el acoplamiento del patrón Observer.
