Promela no es un lenguaje de programación para generar ejecutables (C, Java, Python); es un lenguaje de **especificación y modelado** para describir el comportamiento de procesos concurrentes.

### 1. Tipos de Datos y Simbolismo (`mtype`)

Promela utiliza tipos con tamaños explícitos en bits para mantener acotado el espacio de estados:

- `bit`, `bool` (1 bit)
    
- `byte` (8 bits: `0...255`)
    
- `short` (16 bits)
    
- `int` (32 bits)
    
- `mtype` (Constantes simbólicas): Permite definir etiquetas de texto legibles. **Restricción:** Solo se puede declarar **un único** bloque `mtype` por archivo de Promela.
  
  **Ejemplo:**
  
  mtype = {red, yellow, green};   // Declaración única
  mtype light = green;                  // Instanciación

**Nota sobre `printf`:** El formateador está estrictamente limitado. Para imprimir un valor simbólico de un `mtype`, se debe utilizar obligatoriamente el especificador **`%e`**. Los enteros usan el clásico `%d`.

### 2. Conectores, Operadores y Asignaciones

- `++` y `--` solo se pueden usar como **postfijos** (`i++`) y **nunca** deben combinarse dentro de otras expresiones lógicas o asignaciones.
    
- El operador condicional de C (`cond ? expr1 : expr2`) muta en Promela utilizando una flecha: **`(cond -> expr1 : expr2)`**.
    
- **Las condiciones como sentencias independientes:** En Promela, una expresión evaluativa aislada como `(x > 0);` actúa como una barrera de sincronización. Si es verdadera, el proceso avanza; si es falsa, el proceso **se bloquea inmediatamente** en esa línea hasta que otro proceso altere el valor de la variable global.

### 3. Estructuras de Decisión y Bucles No Deterministas

Tanto en el bloque de selección (`if`) como en el de repetición (`do`), Promela evalúa las condiciones de guarda (`::`). Si más de una condición es verdadera a la vez, SPIN elegirá una de ellas de manera **no determinista** (completamente al azar).

- **Estructura `if` (Sin `else` por defecto):** Si todas las condiciones de un `if` son falsas, el programa no sigue de largo; **se bloquea** esperando que alguna cambie. Para evitarlo, debes mapear explícitamente una rama `:: else -> skip`.
    
    [[ejemplo_if_promela.png]]
    
- **Estructura `do`:** Repite indefinidamente la selección no determinista. La única forma de romper el bucle es emparejar una condición con la instrucción **`break`**.
  
  [[ejemplo_do_promela.png]]

### 4. Ciclo de Vida de los Procesos Concurrentes

Existen dos formas nativas de instanciar y lanzar la ejecución entrelazada (_interleaving_) de procesos en Promela:

- **Instanciación Estática (`active`):** Lanza el proceso directamente al iniciar el entorno. Se puede definir la cantidad exacta de réplicas en paralelo usando corchetes: `active [3] proctype P()`.
    
- **Instanciación Dinámica (`init` y `run`):** Si se define el bloque `init`, este toma el identificador de proceso 0 (`_pid = 0`). Desde dentro de `init`, los procesos parametrizados se lanzan manualmente llamando a la instrucción **`run P('A', 0);`**.

## Comandos de SPIN

SPIN opera bajo dos grandes filosofías: **Simulación** (recorrido aleatorio lineal) y **Verificación Exhaustiva** (exploración matemática completa de la matriz de estados).
### 1. Opciones para el comando de Simulación (`spin [opciones] archivo.pml`)

- `spin archivo.pml` : Ejecuta el modelo de forma lineal tomando decisiones aleatorias en los puntos no deterministas.
    
- `-p` : Muestra en pantalla de forma cronológica todas las declaraciones, sentencias y pasos que van tomando los procesos.
    
- `-l` : Agrega a la traza visible el progreso detallado de las variables locales de cada proceso.
    
- `-u[N]` : (Ej. `-u40`) Establece un **límite duro de N pasos lógicos** (no iteraciones de bucle) para forzar la detención de simulaciones infinitas.
    
- `-t` : **Modo Guiado.** En lugar de simular al azar, obliga a SPIN a leer el archivo `.trail` generado previamente por el verificador exhaustivo, reproduciendo con exactitud el contraejemplo exacto que causó el fallo.

### 2. Opciones para la Verificación Exhaustiva (Model Checking)

Para demostrar de forma matemática absoluta que un sistema está libre de fallas, se genera y compila un analizador específico en C (`pan`).

#### Vía Directa (Sintaxis Compacta)

`spin -search max.pml`

- `-search`: Realiza de forma directa la generación del código del analizador y arranca la búsqueda de errores en una sola llamada integrada.

#### Vía Tradicional por Pasos (Para Control Avanzado)

1. **Generar el código fuente del analizador (`pan.c`):**
	`spin -a max.pml`
	
	- `-a`: Analiza el modelo y genera las estructuras de estados optimizadas dentro de un archivo `pan.c`.
	  
2. **Compilar el binario:**
	`gcc -o pan pan.c -Wformat-overflow=0`
	
	- `-Wformat-overflow=0`: Opción nativa del compilador GCC recomendada para mitigar el volumen excesivo de advertencias de formato generadas por el motor de SPIN.
	  
3. **Ejecutar el analizador personalizado (`./pan [opciones]`):**
	- `./pan` : Ejecuta la búsqueda estándar enfocada en detectar bloqueos (_deadlocks_) o violaciones explícitas de cláusulas `assert`.
	- `-m[N]` : (Ej. `-m70000`) Modifica la profundidad máxima de búsqueda en el árbol de estados (por defecto se trunca estrictamente en el paso 9999).
	- `-E` : **Ignorar Bloqueos.** Ordena al verificador continuar la búsqueda en el resto del árbol aun si detecta estados de _deadlock_ latentes.
	- `-a` : **Búsqueda de Ciclos de Aceptación (Liveness).** Configura el motor para rastrear bucles infinitos no deseados que violen fórmulas de lógica temporal (cuando el sistema se cicla e impide permanentemente alcanzar un estado deseado).

### 3. Verificación de Fórmulas Temporales Avanzadas (`-f`)

Cuando necesitas validar propiedades complejas de exclusión mutua o vitalidad sin ensuciar el código con variables globales de control, inyectas la fórmula LTL directo en el verificador:

`spin -a -f '[]!(P@cs && Q@cs)' archivo.pml`

- **`-f`**: Permite parsear e integrar una fórmula temporal de LTL directamente en la generación del analizador.
    
- **Sintaxis interna de etiquetas:** El operador `Proceso@etiqueta`(ej. `P@cs`) devuelve un valor booleano verdadero si y solo si el proceso indicado se encuentra físicamente posicionado sobre la línea rotulada con dicha etiqueta en ese instante del tiempo.

## Resumen Rápido de Configuración de Ensayos

| **Si quieres verificar...**     | **Comando SPIN inicial**         | **Comando de ejecución de pan** | **Comando de visualización del fallo** |
| ------------------------------- | -------------------------------- | ------------------------------- | -------------------------------------- |
| **Aserciones o Bloqueos**       | `spin -a mi_codigo.pml`          | `./pan`                         | `spin -t -p -1 mi_codigo.pml`          |
| **Propiedades LTL (Vitalidad)** | `spin -a -f '!<>(Q@cs)' cod.pml` | `./pan -a`                      | _Busca la línea `<<START OF CYCLE>>`_  |
| **Sistemas muy profundos**      | `spin -search cod.pml`           | `./pan -m70000`                 | `spin -t -p cod.pml`                   |
