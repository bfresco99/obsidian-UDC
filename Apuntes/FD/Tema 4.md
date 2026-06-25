## Guía 1: Configuración y Ciclo de Vida de Maven

Maven es una herramienta basada en patrones organizativos cuya configuración central se almacena en el archivo **POM (Project Object Model)** en formato XML.
### 1. Inicialización de Datos en el `pom.xml`

Para que Maven identifique un proyecto de forma única, se configuran las denominadas **Coordenadas de Maven** dentro del archivo `pom.xml`:

```
<?xml version="1.0" encoding="UTF-8"?>
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>es.udc.fi.dc</groupId>      <!-- Identificador del grupo (paquete java) -->
    <artifactId>PhotoAlbum</artifactId> <!-- Nombre del proyecto -->
    <version>1.0-SNAPSHOT</version>    <!-- Versión del proyecto (-SNAPSHOT indica desarrollo) -->
    <packaging>war</packaging>          <!-- Formato de empaquetado (jar, war, pom, ear) -->
</project>
```

### 2. Estructura Estándar de Directorios

Maven define una convención estricta para la disposición de los archivos que todo desarrollador debe respetar:

- `src/main/java/`: Archivos de código fuente de la aplicación.
    
- `src/main/resources/`: Archivos de configuración no compilados (propiedades, XML).
    
- `src/main/webapp/`: Archivos de código fuente para entornos web.
    
- `src/test/java/`: Clases y suites de pruebas unitarias.
    
- `target/`: Directorio temporal de trabajo donde se vuelcan los binarios y empaquetados resultantes.

### 3. Ejecución de Fases del Ciclo de Vida

Al invocar una fase en la consola mediante `mvn <fase>`, Maven ejecuta de forma **automática** todas las fases previas del ciclo. Las fases clave en orden secuencial son:

- **`mvn validate`:** Comprueba que el proyecto es correcto y toda la información requerida está disponible.
    
- **`mvn compile`:** Compila el código fuente del proyecto.
    
- **`mvn test`:** Ejecuta las pruebas unitarias usando marcos de trabajo compatibles (como JUnit).
    
- **`mvn package`:** Empaqueta el código binario en su formato distribuible asignado (ej. JAR o WAR).
    
- **`mvn install`:** Installa el artefacto generado dentro del repositorio local de la máquina como caché.
    
- **`mvn deploy`:** Copia el paquete final a un repositorio remoto compartido para el resto del equipo.
    

> **Comando de utilidad combinado:** `mvn clean install` limpia cualquier rastro de compilaciones anteriores y reconstruye el proyecto por completo pasándole las pruebas unitarias e instalándolo localmente.## Guía 1: Configuración y Ciclo de Vida de Maven

## Guía 2: Configuración y Gestión de Tareas en Gradle

Gradle es un sistema de automatización flexible que reemplaza el XML por un **Lenguaje Específico de Dominio (DSL)** basado en Groovy o Kotlin, gestionado a través del archivo `build.gradle`.

### 1. Configuración de Metadatos y Plugins (`build.gradle`)

A diferencia de Maven, la modularización y la carga de comportamientos (como compilar Java) se realizan aplicando plugins directamente en el archivo de configuración:

```
// Aplicación de plugins para extender la funcionalidad del core
apply plugin: 'java'
apply plugin: 'war'

// Definición de metadatos básicos
group = 'com.company'
version = '1.0'
sourceCompatibility = 1.8 // Versión de JRE destino
```

Si el proyecto no sigue la estructura de directorios convencional, se puede reconfigurar explícitamente mediante bloques **`sourceSets`**:

```
sourceSets {
    main { java { srcDir 'src' } } // Cambia la ruta por defecto a una carpeta 'src'
    test { java { srcDir 'test' } }
}
```

### 2. Creación y Ejecución de Tareas Personalizadas

El bloque fundamental de trabajo en Gradle es la **Task (Tarea)**. Cuenta con una fase de configuración (que se evalúa siempre) y una fase de ejecución controlada por bloques imperativos:

```
task hello {
    group 'fancytasks'               // Agrupa la tarea para su organización
    description 'Esta tarea saluda'  // Añade una descripción informativa
    
    doLast {
        println 'Hello Gradle'       // Se ejecuta únicamente al invocar la tarea
    }
}
```

- Para listar todas las tareas del proyecto: `gradle tasks`.
    
- Para ejecutar la tarea creada: `gradle hello`.

### 3. El Gradle Wrapper: Despliegue Consistente

El **Gradle Wrapper** es una de las mejores prácticas recomendadas, ya que permite ejecutar construcciones con una versión de Gradle predefinida y controlada, sin requerir que la máquina o el servidor de CI dispongan de una instalación previa de la herramienta.

- **Generación del Wrapper (v4.9):**
    
    `gradle wrapper --gradle-version 4.9`
    

*   **Uso del Wrapper en lugar del comando global:** Genera scripts adaptados según el entorno de ejecución (`gradlew` para Linux/macOS o `gradlew.bat` para Windows):
    
    `./gradlew build`

## Guía 3: Migración Automática de Maven a Gradle

Si dispones de un entorno de software estructurado con Maven y deseas migrar hacia Gradle aprovechando su rendimiento y flexibilidad, Gradle proporciona una tarea de inicialización en estado de incubación capaz de parsear el archivo XML automáticamente:

1. Posiciónate en la consola de comandos sobre el directorio raíz del proyecto que contiene el archivo `pom.xml`.
    
2. Ejecuta la instrucción de conversión:
    
	`gradle init --type pom`

3.  El motor inspeccionará las dependencias y plugins declarados en Maven y generará los archivos correspondientes `build.gradle` y `settings.gradle` listos para su uso directo.