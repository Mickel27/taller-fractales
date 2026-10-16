# **Informe taller Fractales Modelo Master-Worker**

Miguel Angel Blanco  
Luis Miguel Delgado

# **Pruebas (individuales y grupales)**

Miguel Angel Blanco:

(procesador: AMD Ryzen 7 7435 HS, 8 cores, 16 threads)

| Estrategia | Parámetro | Tareas Generadas | T\_sec (ms) | T\_dist (ms) | Speedup (S) | Eficiencia (E) |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| FixedCount (Franjas) | N \= 2 | 2 | 89271 | 41480 | 2.15x | 107.6% |
| FixedCount (Franjas) | N \= 4 | 4 | 89271 | 23790 | 3.75x | 187.6% |
| FixedCount (Franjas) | N \= 8 | 8 | 89271 | 14153 | 6.31x | 315.4% |
| FixedCount (Franjas) | N \= 16 | 16 | 89271 | 8776 | 10.17x | 508.6% |
| FixedTileSize (Baldosas) | baldosas 32 x 32 | 8160 | 89271 | 8769 | 10.18x | 509.0% |
| FixedTileSize (Baldosas) | baldosas 64 x 64 | 2040 | 89271 | 8557 | 10.43x | 521.6% |
| FixedTileSize (Baldosas) | baldosas 128 x 128 | 510 | 89271 | 8497 | 10.51x | 525.3% |
| FixedTileSize (Baldosas) | baldosas 256 x 256 | 135 | 89271 | 11079 | 8.06x | 402.9% |

Luis Delgado:

Procesador: Intel Core i5 11400H, 6 núcleos y 12 hilos.

Experimento 1:

| Estrategia | Parametro | Tareas\_Creadas | T\_sec\_ms | T\_dist\_ms | Speedup\_S | Eficiencia\_E |
| :---- | :---- | ----- | ----- | ----- | :---- | :---- |
| FixedCount (Franjas fijas) | N \= 2 | 2 | 98374 | 55087 | 1.79x | 89.3% |
| FixedCount (Franjas fijas) | N \= 4 | 4 | 98374 | 32399 | 3.04x | 151.8% |
| FixedCount (Franjas fijas) | N \= 16 | 16 | 98374 | 18335 | 5.37x | 268.3% |
| FixedTileSize (Grano ultrafino) | Baldosas 32x32 | 8160 | 98374 | 19621 | 5.01x | 250.7% |
| FixedTileSize (Grano optimo) | Baldosas 64x64 | 2040 | 98374 | 18510 | 5.31x | 265.7% |
| FixedTileSize (Grano intermedio) | Baldosas 128x128 | 510 | 98374 | 18320 | 5.37x | 268.5% |
| FixedTileSize (Grano grueso) | Baldosas 256x256 | 135 | 98374 | 18166 | 5.42x | 270.8% |

Experimento 2: Concurrencia y Saturación del Pool de Hilos en Workers

| Configuración | Hilos\_ThreadPool | T\_sec\_ms | T\_dist\_ms | Speedup\_S | Eficiencia\_por\_Hilo |
| :---- | ----- | ----- | ----- | :---- | :---- |
| Configuración A (Monohilo) | 1 | 98374 | 114452 | 0.86x | 86.0% |
| Configuración B | 2 | 98374 | 66382 | 1.48x | 74.0% |
| Configuracion C (Cores fisicos) | 6 | 98374 | 25579 | 3.85x | 64.2% |
| Configuracion D (Sobredimensionado) | 64 | 98374 | 13844 | 7.11x | 11.1% |

Experimento 3: Escalabilidad de Thruster y ley de Amdahl

| Nodos\_Workers\_p | Hilos\_Totales | T\_sec\_ms | T\_dist\_ms | Speedup\_Absoluto | Speedup\_Relativo | Eficiencia\_Relativa |
| :---- | ----- | ----- | ----- | :---- | :---- | :---- |
| p \= 1 | 4 | 98374 | 35192 | 2.80x | 1.00x | 100.0% |
| p \= 2 | 8 | 98374 | 18510 | 5.31x | 1.90x | 95.1% |
| p \= 4 | 16 | 98374 | 14331 | 6.86x | 2.46x | 61.4% |

# **Cuestionario**

### **1\. La trampa Espacio vacío vs. zona de Estrés**

Al analizar la naturaleza matemática del conjunto de Mandelbrot, el esfuerzo computacional es altamente heterogéneo. Las regiones externas ("espacio vacío") convergen de manera casi instantánea tras pocas iteraciones, mientras que las áreas cercanas a la frontera del conjunto ("zona de estrés") demandan la cantidad máxima de iteraciones por píxel.  
Las evidencias de las mediciones de Miguel Ángel Blanco demuestran la efectividad de la descomposición en baldosas (FixedTileSize) con el modelo Master-Worker para mitigar la asimetría:

1. **Miguel Ángel Blanco:** Al fragmentar el plano en baldosas de 64x64 (2,040 tareas) y 128x128 (510 tareas), se logran los mejores tiempos de ejecución (8,557 ms y 8,497 ms respectivamente), alcanzando un Speedup superior a 10.4x y eficiencias por encima del 520%. Esto ocurre porque los Workers que terminan rápidamente las baldosas del "espacio vacío" solicitan inmediatamente nuevas tareas del búfer, absorbiendo la carga pesada de las baldosas pertenecientes a la "zona de estrés" sin generar tiempos muertos significativos.  
2. **Luis Delgado:** En la estrategia de baldosas de 128x128 (510 tareas) y 256x256 (135 tareas) se obtienen tiempos de 18,320 ms y 18,166 ms con Speedups de 5.37x y 5.42x, superando con claridad las franjas fijas con N reducidos, donde la asimetría de carga penaliza gravemente la aceleración.

### **2\. Colapso por grano fino vs sobrecosto de Red**

Existe un compromiso intrínseco (trade-off) entre el tamaño de la tarea (granularidad) y el sobrecosto (overhead) asociado al control, sincronización y comunicación en la arquitectura Master-Worker.

- **Análisis de la granularidad**  
  Un tamaño de grano demasiado fino (por ejemplo, baldosas de 32x32 que generan 8,160 tareas) incrementa drásticamente el número de mensajes e interacciones entre el Master y los Workers. Aunque en el entorno local multihilo de Miguel la baldosa de 32x32 ofreció un rendimiento aceptable (8,769 ms vs 8,497 ms en 128x128), en la prueba distribuida de Luis Delgado (Experimento 1\) la baldosa de 32x32 registró un tiempo de 19,621 ms (Speedup 5.01x), resultando más lenta que el grano optimizado de 256x256 (18,166 ms y Speedup 5.42x).  
- **Efecto en redes distribuidas**  
  Cuando el modelo Master-Worker se ejecuta sobre una red física (MPI/Sockets), el sobrecosto de empaquetar, enviar y recibir miles de mensajes pequeños satura el canal de comunicación e incrementa la latencia en la cola de peticiones del Master, haciendo inviable el grano ultrafino.

### **3\. Fracaso de la Partición Estática**

La división en un número fijo de franjas horizontales (FixedCount) constituye una partición estática de la carga. Si bien es muy sencilla de implementar y presenta un sobrecosto de distribución mínimo, resulta ineficiente ante cargas de trabajo no uniformes.

Para N \= 2 franjas, registramos tiempos dist. de 41,480 ms (Speedup 2.15x) y 55,087 ms (Speedup 1.79x). Esto se debe a que la franja central cruza todo el cardioide del fractal (una parte compleja del fractal que requiere alto procesamiento), concentrando la inmensa mayoría del cómputo en un solo hilo/nodo, mientras que la franja superior o inferior finaliza de inmediato, dejando procesadores ociosos.

Al elevar el número de franjas a N \= 16 (8,776 ms en Miguel y 18,335 ms en Luis), la carga se pseudo-dinamiza, acercándose a los tiempos de las baldosas. No obstante, la partición por franjas mantiene una geometría rígida que no se adapta de manera óptima a la topología bidimensional del fractal.

### **4\. Puntos de Quiebre del Hardware y la Memoria**

El rendimiento paralelo no escala indefinidamente con la adición de recursos; está acotado por los límites físicos de la arquitectura del procesador (núcleos físicos vs lógicos) y el ancho de banda del subsistema de memoria.

1. **Impacto de Núcleos Físicos y SMT/Hyper-Threading:** En el procesador de Luis Delgado (6 núcleos / 12 hilos), el Experimento 2 demuestra que pasar de 1 hilo (114,452 ms, Speedup 0.86x) a 6 hilos lógicos (25,579 ms, Speedup 3.85x) genera una ganancia sustancial y eficiente. A partir de los 6 hilos, la escalabilidad pasa a depender del Hyper-Threading, compartiendo unidades de ejecución de punto flotante (ALUs/FPUs), lo que desacelera la ganancia proporcional, es decir, aumenta un poco la eficiencia pero no al mismo paso, al utilizar unidades no dedicadas a estos procesos.  
2. **Sustitución de double por BigDecimal en Altas Magnificaciones:** El tipo primitivo double dispone de 53 bits de mantisa, lo que limita la precisión a \~15-17 dígitos decimales significativos. Al realizar zooms profundos ("altas magnificaciones") en el conjunto de Mandelbrot, el delta entre coordenadas de píxeles adyacentes se vuelve menor que el bit menos significativo representable (es decir, el sistema debe realizar aproximaciones), ocasionando la pérdida de precisión por cancelación catastrófica o subflujo (underflow). La clase *BigDecimal* mitiga este problema al implementar aritmética de precisión arbitraria mediante un valor entero de escala no acotado y una escala de 32 bits, evaluando iteraciones fractales sin acumulación de error de redondeo ni límite de magnitud fija.  
3. **Saturación por Sobredimensionamiento:** En la Configuración D con 64 hilos en la misma máquina de 6 núcleos, el tiempo disminuye a 13,844 ms (Speedup 7.11x), pero la eficiencia por hilo colapsa drásticamente al 11.1%. Esto evidencia la contención severa por el ancho de banda de memoria RAM/caché L3 y el sobrecosto de context switching masivo en el sistema operativo.

### **5\. Límites del ThreadPool y Ley de Amdahl**

El comportamiento de los experimentos distribuidos y multihilo confirma los principios de la Ley de Amdahl y los límites prácticos de los agrupamientos de hilos (ThreadPool):

1. **Fracción Secuencial y Ley de Amdahl:** Toda aplicación posee una fracción secuencial (f), compuesta por la inicialización, la creación/ensamblado de la matriz de imagen y la sincronización final de recepción de resultados por parte del Máster. En el Experimento 3 de Luis (escalamiento con nodos p), aumentar de p=1 (35,192 ms, Speedup rel. 1.00x) a p=2 (18,510 ms, Speedup rel. 1.90x, Eficiencia 95.1%) demuestra una buena escalabilidad lineal, pero al llegar a p=4 nodos (14,331 ms, Speedup real. 2.46x), la eficiencia relativa disminuye al 61.4% debido a que la fracción secuencial de gestión del Master y la red domina el tiempo total de ejecución.  
2. **Comportamiento del ThreadPool:** El ThreadPool reutiliza eficientemente un conjunto fijo de hilos, evitando el sobrecosto de creación y destrucción continua de hilos. No obstante, si el tamaño del pool excede los recursos físicos disponibles, no genera un mayor paralelismo real y la Ley de Amdahl impone una cota máxima infranqueable al aceleramiento total.