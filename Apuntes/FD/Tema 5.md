## 1. Conceptos Clave de Integración Continua (CI)

La **Integración Continua** es una práctica de desarrollo de software donde los miembros de un equipo integran su trabajo de forma frecuente (al menos una vez al día) en un repositorio compartido.

### Beneficios Principales de implementar CI

- **Detección temprana de errores:** Permite identificar fallos de compilación, bugs e incompatibilidades de código de forma inmediata en lugar de hacerlo de manera improvisada en la fecha de lanzamiento.
    
- **Reducción de riesgos en la fusión:** Someter todo el código a pruebas unitarias en tiempo real disminuye drásticamente el riesgo de conflictos complejos durante los _merges_.
    
- **Feedback inmediato:** Proporciona un estado constante y transparente sobre la calidad, funcionalidad e impacto del código nuevo en todo el sistema.
    
- **Disponibilidad del producto:** Asegura que siempre exista un build actualizado y estable listo para pruebas, demostraciones con clientes o lanzamientos a producción.  

### Buenas Prácticas Fundamentales

- **Versionar absolutamente todo:** No basta con meter el código fuente de la aplicación en el SCM; los scripts de pruebas y los archivos de configuración también deben llevar control de versiones para garantizar la trazabilidad.
    
- **Mantener builds rápidos:** El proceso completo de construcción e integración no debería demorar más de **10 minutos**. Si toma más tiempo, fragmenta el flujo dividiéndolo en trabajos paralelos.
    
- **Utilizar una réplica de producción:** El entorno del servidor de CI (sistema operativo, parches de seguridad, librerías del sistema) debe emular con precisión milimétrica el entorno real de producción para evitar el clásico "en mi máquina funciona".

## Guía 1: Configuración de un Build Job en Jenkins

Jenkins es una herramienta basada en Java que se ejecuta sobre contenedores de servlets como Apache Tomcat o de forma independiente por comandos (`java -jar jenkins.war`). Toda su configuración se realiza mediante una interfaz gráfica web.

A continuación, se describen los pasos para configurar un trabajo de construcción e integración para un proyecto basado en Maven:

### Paso 1: Creación del Item

1. En el panel principal de Jenkins, haz clic en **New Item**.
    
2. Asigna un nombre al trabajo (ej: `my-app-integration`).
    
3. Selecciona el tipo de proyecto: **Maven project** (esto optimiza la lectura del archivo `pom.xml` reduciendo los pasos manuales) y presiona **OK**.

### Paso 2: Gestión de Código Fuente (SCM)

1. En la pestaña **Source Code Management**, selecciona la opción **Git**.
    
2. Define la **Repository URL** apuntando a tu servidor (ej: `git@gitlab.fic.udc.es:ferramentas.2023/seton.git`).
    
3. En **Credentials**, asigna la clave SSH privada correspondiente (`Jenkins private SSH key`).
    
4. En **Branch Specifier**, define la rama a vigilar. Usa `*/*` si deseas evaluar de forma abierta cualquier rama o `*/master` para la principal.

### Paso 3: Disparadores de Construcción (Build Triggers)

Define bajo qué condiciones Jenkins compilará automáticamente el código.

1. Selecciona la opción _Build when a change is pushed to GitLab_.
    
2. Configura los disparadores (_triggers_) según las necesidades del equipo:
    
    - **Push Events:** Reacciona ante confirmaciones directas en las ramas.
        
    - **Opened Merge Request Events:** Dispara el proceso de validación en cuanto un desarrollador abre una solicitud de fusión.
        

### Paso 4: Definición del Entorno y Pre-Pasos

1. Si el pipeline depende de dependencias modernas, en **Build Environment** marca la opción _Provide Node & npm bin/ folder to PATH_ seleccionando la versión instalada en el servidor (ej. `node20`).
2. En la sección **Pre Steps**, puedes añadir scripts en Bash (_Execute shell_) para inyectar propiedades dinámicas antes de la compilación o preparar tokens de análisis estático.

### Paso 5: Ejecución del Build y Post-Pasos (Ejecución de Maven)

1. En el campo **Root POM**, confirma el nombre del archivo principal: `pom.xml`.
    
2. En **Goals and options**, ingresa la secuencia de comandos y fases de Maven que se ejecutarán secuencialmente de forma automatizada (ej: `clean install`).
    
3. En la sección **Post Steps**, configura acciones derivadas como lanzar la suite de inspección de código de SonarQube (_Execute SonarQube Scanner_), ingresando las propiedades clave como `sonar.projectKey` y `sonar.projectName`.

## Guía 2: Configuración de Acciones Post-Build (Etiquetado en SCM)

Para garantizar un flujo de trazabilidad completo, es una excelente práctica que, si la integración automatizada resulta exitosa, Jenkins aplique una etiqueta (_tag_) de vuelta al repositorio Git de origen.

1. Abre el menú **Configure** del Job de Jenkins correspondiente.
    
2. Ve a la sección **Post-build Actions** que se ejecuta al terminar todo el pipeline de compilación y pruebas.
    
3. Selecciona la acción **Git Publisher**.
    
4. Configura las siguientes opciones esenciales para resguardar la estabilidad del repositorio:
    
    - Marca el casillero **Push Only If Build Succeeds** (Garantiza que solo el código que superó todas las pruebas unitarias y de integración reciba la etiqueta).
        
    - En el apartado **Tags**, haz clic en _Add Tag_.
        
    - En **Tag to push**, puedes usar variables de entorno dinámicas del servidor para automatizar el versionado, por ejemplo: `${POM_VERSION}_${BUILD_ID}`.

## Guía 3: Integración de Jenkins con Visual Studio Code

Los desarrolladores no necesitan ingresar constantemente al navegador web para auditar el estado del servidor de CI. Utilizando la extensión **Jenkins Jack**, el estado del pipeline se integra de forma directa en el IDE de desarrollo:

1. Instala la extensión **Jenkins Jack** desde el Marketplace oficial de VS Code.
    
2. En la barra lateral del IDE, haz clic sobre el menú de la extensión y selecciona **Jenkins Connections** para añadir un nuevo servidor.
    
3. Sigue el flujo del asistente ingresando los siguientes parámetros de conectividad:
    
    - **Nombre de la conexión:** Define un alias local identificable (ej. `jenkins.fic.udc.es`).
        
    - **URL del Host:** Especifica la dirección completa incluyendo el protocolo de red seguro: `[https://jenkins.fic.udc.es](https://jenkins.fic.udc.es)`.
        
4. Tras autenticarte con tus credenciales de usuario, la barra lateral desplegará en tiempo real:
    
    - El listado de **Jobs** y proyectos configurados.
        
    - El historial cronológico de los últimos _builds_ numerados con códigos de color según su resultado (Verde para éxito, Rojo para fallidos).
        
    - Opción de clic derecho sobre un build fallido para ejecutar **Download Log** y desplegar la consola de salida de Jenkins directamente en una pestaña de tu editor de código para depurar el error rápidamente.