## 1. Virtualización Tradicional vs. Contenedorización

El despliegue de aplicaciones ha evolucionado para solucionar el clásico problema de "en mi máquina funciona" mediante el aislamiento de entornos:

- **Máquinas Virtuales (VM):** Emulan un ordenador físico completo. Cada VM incluye su propio Sistema Operativo invitado (Guest OS) completo, controladores de dispositivos y aplicaciones, corriendo sobre un hipervisor. Esto garantiza un aislamiento total pero consume una enorme cantidad de recursos (CPU, memoria y almacenamiento) y ralentiza los tiempos de arranque.
    
- **Contenedores (Containerization):** Es una forma ligera de virtualización a nivel de sistema operativo. En lugar de emular un hardware y empaquetar un OS entero, los contenedores comparten el mismo kernel del sistema operativo host. Contienen únicamente la aplicación, sus librerías y dependencias necesarias , lo que los hace extremadamente eficientes, rápidos de arrancar y portables entre entornos de desarrollo, pruebas y producción.

## 2. Ecosistema de Docker y Kubernetes

### Docker: El Estándar de Contenedores

Docker permite empaquetar y distribuir aplicaciones mediante tres componentes clave:

1. **Dockerfile:** Archivo de texto plano con instrucciones secuenciales de automatización para construir una imagen de contenedor.
    
2. **Imagen (Image):** Paquete estático, ejecutable y de solo lectura que contiene todo lo necesario para ejecutar la aplicación (código, entorno de ejecución, librerías).
    
3. **Contenedor (Container):** Una instancia viva y en ejecución de una imagen.    

### Kubernetes (K8s): Orquestación a Gran Escala

Cuando una arquitectura de software crece a decenas o cientos de contenedores, gestionarlos manualmente es inviable. Kubernetes actúa como el motor de orquestación automatizada encargado de la distribución, escalado y alta disponibilidad de los contenedores. Sus primitivas principales son:

- **Pod:** La unidad mínima de ejecución en Kubernetes. Aloja uno o más contenedores estrechamente vinculados que comparten almacenamiento y red compartida.
    
- **Deployment:** Define el estado deseado de los Pods (por ejemplo, mantener siempre 3 réplicas idénticas corriendo). Se encarga de sustituir automáticamente los Pods que fallen.
    
- **Service (Servicio):** Una abstracción que define una política de acceso de red estable y balanceo de carga para un grupo de Pods, exponiendo la aplicación hacia el exterior o hacia otros componentes internos.

## Guía 1: Configuración de Empaquetado y Despliegue con Eclipse JKube

**Eclipse JKube** es una colección de plugins de Maven diseñados para simplificar la vida de los desarrolladores Java. Permite compilar la aplicación, generar la imagen de Docker, crear los manifiestos de Kubernetes e inyectarlos en el clúster directamente desde comandos de Maven, sin necesidad de escribir un `Dockerfile` o archivos YAML manualmente.

### 1. Inyección del Plugin en el `pom.xml`

Para habilitar JKube en un proyecto Maven (utilizando la vertiente enfocada a Kubernetes puro), añade el plugin dentro de la sección `<plugins>` del archivo de configuración:

```
<plugin>
    <groupId>org.eclipse.jkube</groupId>
    <artifactId>kubernetes-maven-plugin</artifactId>
    <version>1.16.2</version> </plugin>
```

### 2. Ciclo de Comandos Esenciales de JKube

Una vez añadido, puedes interactuar con tu clúster de Kubernetes ejecutando metas (_goals_) específicas en la consola de comandos de Maven:

- **`mvn k8s:build`:** Compila el código Java, detecta de forma inteligente el tipo de artefacto (ej: un archivo JAR de Spring Boot o un WAR) y construye automáticamente la imagen de Docker en el repositorio local de la máquina.
    
- **`mvn k8s:resource`:** Genera de forma automática los archivos de configuración y manifiestos de Kubernetes (archivos descriptores de _Deployments_ y _Services_) analizando las propiedades del proyecto.
    
- **`mvn k8s:deploy`:** Toma los recursos generados en el paso anterior y los inyecta/despliega de forma directa en el clúster de Kubernetes activo en tu contexto local.
    
- **`mvn k8s:undeploy`:** Elimina de manera limpia y completa todos los recursos, Pods y servicios que fueron creados por el plugin en el clúster.

## Guía 2: Configuración del Pipeline de Despliegue desde Jenkins

En un flujo continuo profesional, el desarrollador no ejecuta los comandos anteriores desde su ordenador local; delega dicha acción en el servidor de Integración Continua (Jenkins).

A continuación se detalla cómo configurar un Build Job en Jenkins para automatizar la compilación, subida a un registro de contenedores e inyección final en Kubernetes utilizando la interfaz gráfica del servidor:

### Paso 1: Configuración General del Job

1. Ingresa a la sección de configuración de tu Job en Jenkins.
    
2. En la sección **Build**, ubica el apartado destinado a los **Goles y opciones de Maven** (_Goals and options_).
    
3. Define la suite secuencial de comandos que limpian, compilan, empaquetan y despliegan la infraestructura de manera integral:
    
    `clean install k8s:build k8s:resource k8s:push k8s:deploy`
    
    - `clean install`: Limpia compilaciones antiguas y empaqueta el nuevo código Java.
        
    - `k8s:build k8s:resource`: Crea la imagen de Docker y genera los YAML de Kubernetes basados en el nuevo empaquetado.
        
    - `k8s:push`: Sube de forma segura la imagen generada hacia un registro central de imágenes corporativo o público (como Docker Hub o un registro privado de la universidad).
        
    - `k8s:deploy`: Aplica los cambios en el entorno de producción o pruebas del clúster de Kubernetes.

### Paso 2: Configuración del Archivo de Credenciales y Ajustes

Para que Jenkins tenga los permisos necesarios para autenticarse contra el registro de contenedores y empujar la imagen (`k8s:push`), requiere acceso a tokens configurados de forma privada:

1. Desplázate hacia la parte inferior del menú de configuración de Maven dentro de Jenkins.
    
2. En el campo **Settings file**, selecciona la opción **Provided settings.xml**.
    
3. En el menú desplegable **Provided Settings**, selecciona tu archivo personalizado de credenciales (ej: `MySettings`), el cual contiene de forma encriptada el usuario y contraseña del registro de Docker.
    
4. Presiona **Save** para activar el pipeline automatizado. A partir de este momento, cada confirmación exitosa de código desplegará un nuevo contenedor actualizado de manera transparente en el clúster orquestado.