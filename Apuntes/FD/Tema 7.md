## 1. Análisis de Rendimiento: Profiling vs. Benchmarking

El documento introduce la optimización de software a través del análisis dinámico de las aplicaciones.

- **Profiling (Perfilado):** Es un análisis dinámico que evalúa de forma pormenorizada el comportamiento de un programa en ejecución (uso de memoria, tiempo de ejecución, frecuencia de instrucciones y duración de llamadas a funciones). Su objetivo primordial es identificar cuellos de botella para optimizar el código.
    
- **Diferencia clave:** A diferencia del _benchmarking_ (que mide el rendimiento general de forma externa), el _profiling_ proporciona mediciones internas de grano fino minimizando los factores ambientales externos. Se realiza instrumentando directamente el código fuente o el binario ejecutable.

## 2. Gestión y Estructura de Memoria en la JVM

Para diagnosticar el rendimiento en Java, es imprescindible entender cómo organiza la JVM la memoria dinámica (Heap).

### Divisiones Principales del Heap

- **Nursery / Young Generation (Zona Joven):** Reservada para la asignación de objetos nuevos. Se subdivide en **Eden**, **Survivor 0 (S0)** y **Survivor 1 (S1)**. Cuando se llena, desencadena una recolección menor (_Minor GC_). Los objetos que sobreviven el tiempo suficiente se promocionan al espacio antiguo.
    
- **Old Space / Memory (Zona Antigua):** Almacena los objetos de larga vida. Cuando se llena, se activa una recolección mayor o completa (_Major / Full GC_) para liberar espacio.
    
- **Non-Heap (Metaspace / PermGen):** Áreas separadas del Heap destinadas a almacenar metadatos de clases, métodos, estructuras internas y pilas de hilos.

### Mecánica de Asignación de Objetos

- **Objetos Pequeños:** Se asignan en áreas locales de hilo llamadas **TLAs (Thread-Local Areas)** dentro de la _nursery_. Esto evita bloqueos globales de hilos aumentando la eficiencia.
    
- **Objetos Grandes:** Si superan el límite establecido para un TLA (usualmente entre 2 y 128 kB), se asignan **directamente** en el _Old Space_.

### Gestión Automática de Memoria y Fugas (Memory Leaks)

El uso de un recolector de basura (_Garbage Collector_) evita errores clásicos de gestión manual como los punteros colgantes (_dangling references_) y pérdidas de espacio. Sin embargo, asumir que el GC lo resuelve todo puede hacer que los desarrolladores descuiden las **fugas de memoria** (objetos retenidos en el código que ya no son útiles). Esto satura el Heap, provocando lentitud por intercambio de disco (_disk swapping_) o excepciones críticas de tipo **`OutOfMemoryError`**.

## Guía de Uso: Java Mission Control (JMC) y Flight Recorder (JFR)

**Java Mission Control (JMC)** es una herramienta avanzada y no intrusiva diseñada para recolectar información detallada a bajo nivel de la JVM en tiempo de ejecución, lo que la hace completamente viable para entornos de producción. **Java Flight Recorder (JFR)** es el framework interno encargado de capturar dichos eventos.

### 1. Conexión e Inspección en Tiempo Real (Consola JMX)

1. Al abrir JMC, el panel **JVM Browser** listará todos los procesos Java activos en la máquina.
    
2. Haz doble clic sobre el proceso deseado o su **MBean Server** para abrir el cuadro de mandos interactivo (_Dashboard_).
    
3. **Vistas en Tiempo Real:**
    
    - **Overview:** Ofrece relojes e indicadores visuales instantáneos sobre el uso del Heap, uso de CPU de la JVM y fragmentación.
        
    - **MBeans Browser:** Permite inspeccionar el árbol completo de objetos gestionados (como `GarbageCollector` o `MemoryPool`), exponiendo sus atributos y operaciones ejecutables en vivo.
        
    - **Triggers (Disparadores):** Pestaña crítica donde se pueden configurar reglas automáticas. Se puede programar para que, si el conteo de hilos es muy alto (`Thread Count Too High`) o la CPU se satura, el sistema envíe un correo, registre un log o dispare un volcado de memoria automáticamente.

### 2. Configuración y Lanzamiento de una Grabación (Flight Recording)

Para registrar datos históricos exactos y analizarlos tras un incidente, se debe iniciar una grabación de JFR:

1. En el _JVM Browser_, haz clic derecho sobre **Flight Recorder** y selecciona **Start Flight Recording**.
    
2. **Parámetros de Grabación:**
    
    - **Time fixed recording:** Define una duración limitada (ej. `1 min`) para capturar un comportamiento concreto.
        
    - **Continuous recording:** Graba continuamente guardando en disco los límites máximos de tamaño o antigüedad definidos.
        
3. **Ajustes de Eventos (Event Options):** Permite configurar el nivel de detalle del perfilado. Es recomendable activar **`Allocation Profiling`** para rastrear objetos y modificar el muestreo de métodos (**`Method Sampling`**) a nivel _Maximum_ para auditorías estrictas de CPU. Haz clic en _Finish_.

### 3. Diagnóstico Técnico de la Grabación (Vistas de Análisis)

Una vez completada la grabación, se abrirá un archivo estructurado conteniendo los siguientes paneles especializados para auditoría:

- **Memory View (Memoria):** Muestra gráficas temporales de la memoria reservada frente a la usada. Abajo incluye pestañas esenciales como:
    
    - **Garbage Collections:** Muestra cuántas recolecciones ocurrieron, sus motivos, duraciones e identificadores únicos.
        
    - **Allocations:** Entrega estadísticas detalladas de los buffers TLAB creados frente a las asignaciones de objetos gigantes hechas directamente fuera de los TLAB en la memoria antigua.

- **Code View (Código):** Localiza con precisión qué secciones de la aplicación degradan el rendimiento.
    
    - **Hot Packages / Hot Classes:** Gráficos de tarta y tablas que ordenan los paquetes y clases según la cantidad de muestras de CPU en las que estuvieron activos.
        
    - **Hot Methods:** Lista los métodos específicos que más tiempo de procesador consumieron.
        
    - **Call Tree (Árbol de llamadas):** Despliega las trazas completas de ejecución (_Stack Traces_) en formato de jerarquía arbórea para saber exactamente qué flujos invocaron a esos métodos lentos.
        
- **Threads View (Hilos):** Centraliza la monitorización de la concurrencia.
    
    - **Latencies:** Expone las causas de esperas en los hilos (ej. bloqueos por sincronización en `java.lang.Object.wait` o lecturas de red con `Socket Read`).
        
    - **Thread Dumps:** Proporciona instantáneas completas de texto del estado interno de cada hilo para detectar bloqueos mutuos (_deadlocks_).
        
- **I/O View (Entrada/Salida):** Muestra las rutas de archivos físicas (`Path`) y sockets con los que la aplicación ha interactuado, indicando el tiempo de escritura/lectura y los bytes totales transferidos.