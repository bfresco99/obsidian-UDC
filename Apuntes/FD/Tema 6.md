## 1. Conceptos de Calidad de Software e Inspección Continua

La calidad del software es un aspecto crítico en los sistemas de negocio modernos, donde el mantenimiento representa aproximadamente el **80% del coste total** del ciclo de vida de un producto de software. Esta calidad se divide en dos grandes vertientes:

- **Calidad Externa (Funcional):** Evalúa el grado en el que el programa cumple con los requisitos y expectativas directas del cliente o usuario final.
    
- **Calidad Interna (Técnica):** Se enfoca en las características estructurales del código, tales como su robustez, legibilidad, cumplimiento de estándares de programación y, sobre todo, su **mantenibilidad**.

### El Cambio de Paradigma: Auditorías vs. Inspección Continua

- **Métodos Tradicionales:** Se basan en auditorías puntuales o controles periódicos al final de una fase de desarrollo. Esto suele provocar retrasos en las entregas, la necesidad de rehacer grandes partes del código (_rework_) o la liberación de software con baja calidad estructural.
    
- **Inspección Continua:** Es un proceso exhaustivo que integra la evaluación de la calidad interna directamente en el ciclo de vida del desarrollo de software. Cada vez que un desarrollador modifica el código, este se analiza de manera automatizada para asegurar una retroalimentación inmediata sobre la deuda técnica acumulada.

## Guía 1: Configuración de SonarQube (Inspección en el Servidor)

**SonarQube** es la plataforma centralizada que recopila las métricas de calidad de los proyectos. El análisis se ejecuta habitualmente de manera automatizada integrándose con los trabajos (_jobs_) de un servidor de Integración Continua como Jenkins.

Una vez finalizado el análisis mediante el escáner (`EXECUTION SUCCESS`), el reporte se empaqueta y sube automáticamente al servidor de SonarQube, generando un enlace directo al panel de control (_dashboard_) del proyecto:

```
https://sonar.fic.udc.es/dashboard?id=es.udc.fi.dc.fd%3Aseton&branch=master
```

### El Paradigma de los "Clean Pools" (Métricas Clave de Calidad)

Al ingresar al panel de SonarQube, la herramienta evalúa el código bajo las siguientes clasificaciones de deficiencias:

1. **Bugs:** Fallos funcionales o de fiabilidad que pueden causar un comportamiento erróneo o una caída del sistema en producción.
    
2. **Vulnerabilidades:** Brechas de seguridad reales o potenciales que exponen al sistema a ataques externos.
    
3. **Code Smells (Malos olores de código):** Deficiencias de mantenibilidad que, aunque no impiden el funcionamiento del programa, complican su lectura, reutilización o modificación futura (generando deuda técnica).
    
4. **Copy-Paste Detector (CPD):** El motor calcula automáticamente el porcentaje de duplicación de líneas de código en los archivos del proyecto para evitar la redundancia y facilitar el mantenimiento centralizado.

## Guía 2: Configuración de SonarLint en VS Code (Modo Conectado)

**SonarLint** funciona como un corrector ortográfico de código integrado directamente en el IDE del desarrollador. Al configurarlo en **Connected Mode**, las reglas de análisis de la suite local se sincronizan directamente con los estándares y perfiles de calidad definidos de manera centralizada en el servidor corporativo de SonarQube.

### Paso 1: Configurar el Archivo de Ajustes Locales (`settings.json`)

Para vincular tu entorno local de Visual Studio Code con el servidor de la universidad o empresa, debes añadir la URL del servidor dentro de la configuración de la extensión:

```
"sonarlint.connectedMode.connections.sonarqube": [
    {
        "connectionId": "https-sonar-fic-udc-es",
        "serverUrl": "https://sonar.fic.udc.es"
    }
]
```

### Paso 2: Vincular el Proyecto Específico

Para indicarle a la extensión qué proyecto del servidor corresponde a tu espacio de trabajo local, se crea o edita el archivo de configuración de SonarLint dentro de la carpeta oculta `.vscode/sonarlint.json`:

```
{
    "connectionId": "https-sonar-fic-udc-es",
    "projectKey": "es.udc.fi.dc.fd:imagination"
}
```

### Paso 3: Auditoría y Depuración Directa en el Editor

Una vez establecido el Modo Conectado (_Connected Mode_), el desarrollador obtiene los siguientes beneficios en tiempo real:

- **Alertas Tempranas:** Al abrir cualquier archivo (ej: `SecurityConfig.java`), la pestaña **Problems** de VS Code desplegará inmediatamente las advertencias de código detectadas de forma interactiva (ej: _"Inject this field value directly"_ o _"Make sure that enabling CORS is safe here"_).
    
- **Documentación Integrada:** Al hacer clic sobre cualquier problema listado, se abre la pestaña **Documentation**, la cual explica detalladamente la regla infringida, por qué se considera una mala práctica y expone ejemplos de código erróneo frente a la solución recomendada (_Noncompliant_ vs _Compliant_).

