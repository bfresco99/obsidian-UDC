## 1. Fundamentos de las Pruebas de Rendimiento

Las pruebas de rendimiento son una categoría crucial de las **pruebas no funcionales** cuyo objetivo no es validar _qué_ hace el sistema, sino _cómo_ opera bajo cargas de trabajo específicas, midiendo atributos clave como la capacidad de respuesta, estabilidad, escalabilidad y eficiencia de recursos.

### Tipología de Pruebas de Rendimiento

- **Load Testing (Pruebas de Carga):** Evalúa el comportamiento del sistema bajo una carga de trabajo esperada (usuarios concurrentes y volumen de transacciones normales) para identificar cuellos de botella de software y hardware.
    
- **Soak / Endurance Testing (Pruebas de Resistencia):** Aplica una carga sostenida durante un periodo prolongado con el fin de detectar **fugas de memoria (memory leaks)** y prevenir la degradación del rendimiento en el tiempo.
    
- **Spike Testing:** Somete al sistema a incrementos y decrementos de carga repentinos y masivos para verificar si es capaz de adaptarse de forma abrupta sin colapsar.
    
- **Stress Testing (Pruebas de Estrés):** Empuja el sistema más allá de sus límites máximos esperados para evaluar su robustez bajo condiciones extremas y ver cómo se recupera ante un fallo masivo.
    
- **Configuration Testing:** Varía la configuración de los componentes de la infraestructura (por ejemplo, algoritmos de balanceo de carga) en lugar de la cantidad de usuarios, buscando optimizar el setup técnico.

## Guía 1: Configuración de Escenarios de Carga en Apache JMeter

Apache JMeter es una herramienta portátil 100% Java diseñada para simular cargas pesadas sobre recursos estáticos y dinámicos. **Nota crítica:** JMeter actúa a nivel de protocolo, por lo que **no es un navegador** (no renderiza HTML ni ejecuta JavaScript).

### Paso 1: Configurar el Grupo de Hilos (Thread Group)

1. Haz clic derecho sobre el _Test Plan_ vacío → **Add** → **Threads (Users)** → **Thread Group**.
    
2. **Number of Threads (users):** Define la cantidad de usuarios virtuales simultáneos.
    
3. **Ramp-Up Period (seconds):** Tiempo total que tardará JMeter en encender a todos los hilos. Por ejemplo, para 100 usuarios y un Ramp-Up de 50 segundos, el sistema iniciará 2 hilos nuevos cada segundo (50/25). Un valor de `0` enciende a todos a la vez.

### Paso 2: Centralizar Variables Globales (HTTP Request Defaults)

Para evitar reescribir IPs y puertos en cada petición:

1. Clic derecho sobre el _Thread Group_ → **Add** → **Config Element** → **HTTP Request Defaults**.
    
2. Rellena los campos **Server Name or IP** (ej: `127.0.0.1`) y **Port Number** (ej: `8080`).

### Paso 3: Gestión de Sesión y Datos Dinámicos (Cookies y CSRF)

- **HTTP Cookie Manager:** Añádelo para que cada hilo almacene y transmita de forma independiente sus cookies (manteniendo sesiones activas simuladas).
    
- **Regular Expression Extractor (Post-Processor):** Si una petición POST (como un inicio de sesión) requiere un token de seguridad que cambia dinámicamente, añádelo bajo la petición GET de la página inicial.
    
    - _Reference Name:_ `csrf_token`
        
    - _Regular Expression:_ `name="_csrf" value="(.+?)"`
        
    - _Template:_ `$1$`
        
    - En las peticiones POST subsecuentes, invoca el valor dinámico usando la sintaxis `${csrf_token}`.

### Paso 4: Carga Masiva de Credenciales (CSV Data Set Config)

1. Añade un componente **CSV Data Set Config**.
    
2. **Filename:** Ruta física del archivo (ej: `/home/fd/users.csv`).
    
3. **Variable Names:** Nombres de las columnas separados por comas (ej: `user,pass`). En los parámetros de login de tu Sampler HTTP, reemplaza los valores de texto por `${user}` y `${pass}`.

### Paso 5: Ejecución Confiable por Línea de Comandos (CLI)

**Regla de oro:** El entorno gráfico (GUI) de JMeter consume excesivos recursos de la máquina e invalida las métricas. Genera y depura tu script en la GUI, pero ejecútalo en producción usando la terminal:

```
jmeter -n -t miTest.jmx -l log.jtl -e -o ./results
```

- `-n`: Ejecución sin entorno gráfico (Non-GUI).
    
- `-t`: Ruta del archivo del plan de pruebas (`.jmx`).
    
- `-l`: Nombre del archivo de salida de logs crudos.
    
- `-e -o`: Genera automáticamente una carpeta web interactiva con el reporte HTML de métricas funcionales tras el test.

## Guía 2: Pruebas de Carga Modernas y Solares con Grafana k6

**Grafana k6** es una herramienta programable de alto rendimiento escrita en Go que utiliza scripts basados en **JavaScript** para estructurar escenarios complejos de microservicios y APIs.

### 1. Estructura de Ciclo de Vida y Pruebas

Los scripts en k6 se configuran respetando estrictamente 4 etapas secuenciales continuas:

```
import http from 'k6/http';
import { sleep } from 'k6';

// 1. INIT CONTEXT: Configuración global, carga de archivos locales e importaciones.
export const options = {
  stages: [
    { duration: '30s', target: 20 },  // Rampa ascendente: sube a 20 usuarios en 30s.
    { duration: '1m30s', target: 10 }, // Baja y mantiene a 10 usuarios en 1m 30s.
    { duration: '20s', target: 0 },   // Rampa descendente: limpia a 0 usuarios.
  ],
  thresholds: {
    http_req_failed: ['rate<0.01'],   // Criterio de fallo: menos del 1% de errores permisibles.
    http_req_duration: ['p(95)<200'], // El 95% de las peticiones deben tardar menos de 200ms.
  },
};

// 2. SETUP (Opcional): Prepara el entorno o genera tokens de acceso.
export function setup() {
  // Retorna datos necesarios para las VUs
  return { token: 'secret-auth-key' };
}

// 3. VU CODE (Obligatorio): Lógica central del usuario virtual, se itera en bucle.
export default function (data) {
  const url = 'http://test.k6.io/login';
  const payload = JSON.stringify({ email: 'user@udc.es', password: 'abc' });
  const params = { headers: { 'Content-Type': 'application/json' } };

  http.post(url, payload, params);
  sleep(1); // Simula el tiempo de lectura o pensamiento del usuario humano.
}

// 4. TEARDOWN (Opcional): Procesa resultados remotos o destruye datos de prueba.
export function teardown(data) {
  console.log('Test finalizado de forma correcta.');
}
```

### 2. Comandos de Ejecución y Exportación Externa

k6 permite mutar sus entornos de ejecución de forma nativa sin modificar el script core:

- **Ejecución Local Rápida (Sobrescribiendo opciones):**
    
    `k6 run --vus 50 --duration 1m script.js`
    
- **Exportación a sistemas de monitorización (Time Series):** Puedes enviar las métricas crudas en tiempo real de cada petición directamente hacia plataformas analíticas de infraestructura usando el flag `--out`:
    
    `k6 run --out json=reporte.json --out influxdb=http://localhost:8086/k6db script.js`

### 3. Visualización en Tiempo Real: El Web Dashboard Integrado

k6 cuenta con una interfaz web integrada nativa para monitorizar las métricas sin depender de configuraciones adicionales de bases de datos externas:

1. Lanza tu script levantando la variable de entorno del panel de control:
    
    `K6_WEB_DASHBOARD=true k6 run script.js`
    
2. Abre tu navegador de preferencia e ingresa a la URL local por defecto:
    
    `http://127.0.0.1:5665`
    
3. **Exportación Estática:** Si estás corriendo las pruebas dentro de una tubería de CI/CD y requieres guardar un informe visual en HTML para auditarlo posteriormente, añade la variable de exportación:
    
    `K6_WEB_DASHBOARD=true K6_WEB_DASHBOARD_EXPORT=resultado_rendimiento.html k6 run script.js`