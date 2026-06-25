## 1. Introducción a los Issue Trackers

Los proyectos de software a gran escala requieren herramientas automatizadas para gestionar tareas y realizar el seguimiento de problemas de manera eficiente.

- **¿Qué es un ITS?:** Es una aplicación de software que utilizan las organizaciones para monitorear y mantener una lista de problemas reportados (también conocidos como sistemas de tickets).
    
- **¿Qué es un Ticket?:** Es un registro dentro del sistema que contiene información detallada sobre un problema específico, respondiendo a preguntas clave como:
    
    - ¿Quién es el responsable de resolverlo?
        
    - ¿Qué características se incluyen en una versión?
        
    - ¿Dónde está la documentación de cada versión?
        
    - ¿Cómo se reporta un fallo (bug)?

## 2. Buenas Prácticas en el Seguimiento de Problemas

Para optimizar el uso de un ITS se recomiendan las siguientes pautas:

- **Mantenerlo simple:** El proceso para reportar defectos y abrir tareas debe ser intuitivo para todos los usuarios y partes interesadas.
    
- **Mostrar información en el punto de decisión:** Actualizar los datos a medida que progresa el problema. Integrar estos detalles directamente en el IDE del desarrollador aumenta significativamente la productividad.
    
- **Usar terminología relevante:** Asegurar que todo el equipo utilice y conozca términos claros y consistentes.
    
- **Capturar datos del reporte:** Incluir datos esenciales (fechas, información del usuario, detalles de la organización, tipo de problema, etc.).
    
- **Escribir reportes claros y reproducibles:** Facilitar título, resumen, configuración del sistema, pasos exactos para reproducir el fallo y resultados esperados.
    
- **Reducir la ambigüedad con capturas:** Adjuntar capturas de pantalla para ilustrar errores o maquetas de interfaz para nuevas funciones.
    
- **Evitar la duplicación:** Comprobar si un fallo ya está registrado antes de crear un nuevo reporte.
    
- **Adaptarse al flujo de trabajo del equipo:** Seguir un ciclo de vida claro: Crear $\rightarrow$ Priorizar/Planificar $\rightarrow$ Asignar recursos $\rightarrow$ Implementar/Liberar $\rightarrow$ Cerrar.
    
- **Integración con la gestión de cambios:** Vincular el sistema de gestión de problemas con los sistemas de control de versiones (SCM) para un rastreo continuo.    

> **Ejemplos de ITS comunes:** Bugzilla, Jira, Trac, Redmine/ChiliProject, Lighthouse, Sifter, entre otros.

## 3. Redmine: Características y Gestión de Proyectos

**Redmine** es una herramienta web versátil para la gestión de proyectos cuyo elemento central es el **Issue** (asunto/problema), el cual representa una tarea específica y accionable (no limitada a la programación). Ayuda a dividir proyectos grandes, organizar tareas, rastrear progresos, resolver fallos y mejorar la comunicación corporativa.

### Características Principales (Pestañas de Redmine)

- **Activity (Actividad):** Vista general de los cambios y actualizaciones recientes del proyecto.
    
- **News (Noticias):** Espacio destinado a anuncios importantes para mantener informado al equipo.
    
- **Forums (Foros):** Lugar de discusión y documentación colaborativa para los interesados.
    
- **Wiki:** Espacio para descripciones del proyecto, decisiones de diseño, planes de futuro y documentación complementaria.
    
- **Documents (Documentos):** Permite subir y organizar la documentación técnica o del proyecto por categorías.
    
- **Files (Archivos):** Repositorio de archivos asociados al proyecto (parches, imágenes, documentos de gestión, etc.) con control de hash MD5.
    
- **Repository (Repositorio):** Conexión con sistemas SCM (como Git) para examinar el historial del código.
    
- **Time Tracking (Control de tiempo):** Herramienta para registrar y auditar las horas invertidas en las tareas.

## 4. Gestión de Asuntos (Issues) en Redmine

Al crear o gestionar un issue en Redmine, se configuran múltiples atributos que definen su ciclo de vida:

### Atributos de un Asunto

- **Tracker (Seguimiento):** Define el tipo de problema. Por defecto se incluyen **Errores (Errors)**, **Tareas (Tasks)** y **Soporte (Support)**.
    
- **Status (Estado):** Estado actual del ticket. Por defecto: **Nueva (New)**, **En curso (Active)**, **Resuelta (Solved)**, **Comentarios (Comments)**, **Cerrada (Closed)** y **Rechazada (Rejected)**.
    
- **Priority Levels (Prioridad):** Severidad de la tarea: **Baja (Low)**, **Normal**, **Alta (High)**, **Urgente (Urgent)** e **Inmediata (Immediate)**.
    
- **Assignee (Asignado a):** Miembro del equipo responsable de resolver el problema.
    
- **Target Version (Versión objetivo):** Hito o versión del software en la cual el problema debe estar resuelto o cerrado.
    
- **Parent Task (Tarea padre):** Permite jerarquizar tareas vinculando subasuntos a un problema principal.
    
- **Detalles adicionales:** Fecha de inicio, fecha de vencimiento, tiempo estimado, porcentaje de realización, archivos adjuntos y observadores (watchers).
### Vistas y Reportes

Redmine ofrece diferentes formatos para visualizar el estado de las tareas:

1. **Vista de Lista:** Muestra los filtros aplicados y las columnas del estado de los issues.
    
2. **Summary (Resumen):** Gráficas y tablas que ofrecen un análisis global de aspectos como asignados, autores, categorías y versiones.
    
3. **Calendar (Calendario):** Distribución diaria de las tareas y plazos de entrega.
    
4. **Gantt:** Cronograma tradicional interactivo para visualizar las líneas de tiempo del proyecto.
    
5. **My Page (Mi Página):** Panel personalizado que centraliza los problemas asignados y reportados por el usuario actual.

## 5. Integraciones Avanzadas (SCM y VS Code)

### Integración con Control de Versiones (Git)

Incluso sin complementos complejos, los mensajes de confirmación (_commit_) de Git pueden interactuar directamente con Redmine mediante palabras clave para actualizar estados o registrar tiempos automáticamente:

- Asociar y cerrar problemas: `git commit -m "...this commit refs #1, #2 and fixes #3"`
    
- Cerrar un ticket específico: `git commit -m "...this commit closes #3"`
    
- Registrar tiempo dedicado: `git commit -m "implements feature #5 @3h15m"`
    

### Integración con Visual Studio Code

Mediante extensiones dedicadas, los desarrolladores pueden conectar su entorno de programación con el servidor de Redmine utilizando el **Identificador del Proyecto**, la **URL** y la **API Key** personal.

- Permite ver la lista de tareas asignadas y proyectos directamente en la barra lateral del editor.
    
- Permite hacer clic sobre un ticket para **cambiar su estado**, **añadir comentarios** o **abrirlo en el navegador** sin salir de VS Code.
    
- Facilita el **registro de tiempos** (_Time Logging_), seleccionando el tipo de actividad (Diseño, Desarrollo, Análisis, etc.), indicando las horas y añadiendo una descripción.