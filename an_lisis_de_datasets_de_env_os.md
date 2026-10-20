# Análisis Detallado de los Datasets

---

## 1. Estructura General del Dataset

Cada archivo CSV contiene 15 variables que describen el paquete, las condiciones del entorno y la clasificación del vehículo asignado:

---

## 2. Análisis de cada uno de los archivos

### 1. `envios1.csv` — Dataset Base (Línea de Control)
* **Perfil del lote:** Distribución estándar y balanceada de todas las variables.
* **Comportamiento del sistema:** Las asignaciones de `transporte_sugerido` siguen patrones intuitivos y proporcionales. Distancias cortas ($<5\text{ km}$) con pesos bajos ($<2\text{ kg}$) se dirigen a `bicicleta` o `dron`, mientras que distancias largas o volúmenes mayores se asignan a `camioneta`.

### 2. `envios2.csv` — Carga de Gran Volumen y Peso
* **Perfil del lote:** Concentración de valores altos en `peso_kg`, `volumen_l` y dimensiones físicas (`largo_cm`, `ancho_cm`, `alto_cm`).
* **Comportamiento del sistema:** Inhabilitación automática de vehículos ligeros (`bicicleta`, `dron`) por exceso de peso/volumen. Predominio masivo del tipo de vehículo `camioneta`.

### 3. `envios3.csv` — Entorno Urbano y Congestión Vial
* **Perfil del lote:** Alta prevalencia de zonas `centro` y `residencial`, combinadas con índices elevados en `trafico_nivel`.
* **Comportamiento del sistema:** El sistema penaliza los vehículos de cuatro ruedas (`camioneta`) por riesgo de retraso en tráfico denso, favoreciendo opciones ágiles como `moto` y `dron` para paquetes pequeños, y `bicicleta` en distancias cortas.

### 4. `envios4.csv` — Restricciones Climáticas y Zonas Periféricas
* **Perfil del lote:** Alta frecuencia de zonas `rural` y `periferia`, acompañadas de condiciones climáticas adversas como `lluvia` y `viento_fuerte`.
* **Comportamiento del sistema:** Las variables de clima actúan como anuladores de seguridad. La presencia de `viento_fuerte` descarta automáticamente el uso de `dron`, mientras que la `lluvia` reduce drásticamente el uso de `bicicleta`, desviando el tráfico a `moto` equipada o `camioneta`.

### 5. `envios5.csv` — Cadena de Frío, Fragilidad y Alto Valor
* **Perfil del lote:** Proporción elevada de registros con `refrigerado = 1`, `fragil = 1` y valores altos en `valor_declarado_mxn`.
* **Comportamiento del sistema:** Restricción de vehículos abiertos (`bicicleta`, `dron`) debido a la imposibilidad de mantener control térmico o asegurar mercancía de alto valor contra robos/daños. La `camioneta` domina la asignación en esta categoría.

### 6. `envios6.csv` — Escenarios de Estrés y Alta Prioridad
* **Perfil del lote:** Densidad elevada de entregas con prioridad `mismo_dia` y `express`, entrelazadas con condiciones adversas combinadas.
* **Comportamiento del sistema:** Pone a prueba la resolución de conflictos cuando chocan restricciones opuestas (ej. alta urgencia + alto tráfico + lluvia + paquete pesado).

---

## 2. Análisis de Mezcla y Complejidad Multivariable

La información contenida en los datasets **no es homogénea ni ordenada linealmente**, lo que incrementa el nivel de realismo y complejidad:

1. **Ausencia de Orden Secuencial:** Los registros dentro de cada archivo aparecen de forma aleatoria. Por ejemplo, en `envios1.csv` un paquete de $1.4\text{ km}$ precede a uno de $49\text{ km}$, mezclando distintos tipos de zona y clima en filas contiguas.
2. **Entropía e Intercalación de Anomalías:** Ningún archivo es "puro". Aunque `envios5.csv` está centrado en productos frágiles y refrigerados, contiene filas intercaladas con `refrigerado = 0` y `fragil = 0`, evitando que el modelo pueda sobreajustarse a reglas triviales.
3. **Cruce Contradictorio de Variables:** Ocurren situaciones borde, como paquetes con muy bajo peso ($<1\text{ kg}$) pero etiquetados como `mismo_dia` bajo `viento_fuerte`, donde el peso sugiere `dron`, la urgencia sugiere `moto`, pero el clima prohíbe el `dron`.

