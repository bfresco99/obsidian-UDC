## Guía de Configuración y Comandos Esenciales de Git

### 1. Configuración Inicial del Entorno

Antes de interactuar con cualquier repositorio, es necesario establecer la identidad global en la máquina de desarrollo. Esto asegura que cada confirmación de código quede registrada con el autor correcto.

- **Configurar el nombre de usuario:**
  `git config --global user.name "Tu Nombre"`
- **Configurar el correo electrónico:**
  `git config --global user.email "tu_correo@udc.es"`
- **Verificar la configuración activa:**
  `git config --list`

### 2. Creación y Clonación de Repositorios

Existen dos formas estándar de iniciar un espacio de trabajo controlado por Git:

- **Inicializar un repositorio local desde cero:** Transforma un directorio local existente en un repositorio de Git (creando la carpeta oculta `.git`).
  
  `git init`
  
- **Clonar un repositorio remoto existente:** Descarga una copia completa (historial y archivos) de un proyecto alojado en servidores como GitHub, GitLab o un servidor propio.
  
  `git clone <url_del_repositorio>`

### 3. El Flujo de Trabajo Local (Las 3 Áreas de Git)

El núcleo operativo de Git se gestiona a través de la transición de los archivos por tres zonas de control locales:

1. **Working Directory (Directorio de Trabajo):** El árbol de archivos físicos en tu disco duro donde creas, editas o eliminas código de forma activa.
    
2. **Staging Area / Index (Área de Preparación):** Una zona intermedia donde se preparan y agrupan los cambios específicos que van a formar parte de la próxima instantánea del proyecto.
    
3. **Local Repository (Repositorio Local):** La base de datos interna donde se almacenan permanentemente las confirmaciones con su respectivo historial.

`[ Directorio de Trabajo ] --( git add )--> [ Área de Staging ] --( git commit )--> [ Repositorio Local ]`

### 4. Ciclo de Comandos para el Control de Cambios

Para mover archivos a través de las áreas mencionadas y mantener el repositorio al día, se ejecuta el siguiente ciclo de comandos básicos:

- **Comprobación del estado:** Muestra qué archivos han sido modificados, cuáles están listos en el área de preparación (_staging_) y cuáles no están siendo rastreados por Git.
  
  `git status`

- **Preparar los cambios (Añadir al Index):** Mueve los archivos modificados o nuevos hacia el área de staging.
  
  `git add <nombre_del_archivo>`
  `git add .`

- **Confirmar los cambios (Crear un Commit):** Consolida de manera permanente los cambios del área de staging en el repositorio local. **Siempre** se debe incluir un mensaje descriptivo.
  
  `git commit -m "Explicación clara del cambio introducido"`

### 5. Sincronización con Repositorios Remotos

Una vez que has estructurado e implementado tus _commits_ a nivel local, es momento de sincronizar el trabajo con el resto del equipo:

- **Subir cambios locales al servidor remoto:** Envía las confirmaciones guardadas en tu base de datos local hacia la rama correspondiente del servidor remoto (ej. _origin_).
  
  `git push origin <nombre_de_la_rama>`

- **Descargar y fusionar cambios del servidor:** Trae las últimas actualizaciones hechas por otros desarrolladores en el repositorio remoto y las combina automáticamente con tu copia de trabajo local.
  
  `git pull`
