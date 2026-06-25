## 1. Fundamentos de las Pruebas Funcionales

Las pruebas funcionales verifican que una característica o componente específico de un sistema de software funcione según lo previsto, comparando su salida real con los resultados esperados basados en las especificaciones.

- **Enfoque estructural:** Se centran en los **requisitos funcionales** (_qué_ hace el sistema) , en contraste con las pruebas no funcionales que evalúan cualidades como el rendimiento o la usabilidad (_cómo_ opera).
    
- **Nivel de granularidad:** El documento define tres niveles esenciales para el ciclo de vida de desarrollo:
    
    1. **Unit Tests (Pruebas Unitarias):** Evalúan la unidad más pequeña de funcionalidad aislada, típicamente métodos o funciones individuales.
        
    2. **Integration Tests (Pruebas de Integración):** Verifican la interacción y comunicación correcta entre múltiples componentes o subsistemas combinados.
        
    3. **Acceptance Tests (Pruebas de Aceptación):** Validan el sistema completo frente a los requisitos del cliente o usuario final para autorizar su entrega.

## Guía 1: Configuración de Pruebas Unitarias y Cobertura en Maven (Java)

Para entornos Java, la automatización de la compilación y ejecución de pruebas funcionales se apoya en plugins especializados dentro del archivo central `pom.xml`.

### 1. Separación de Ciclos de Vida: Surefire vs. Failsafe

Maven delega la ejecución de pruebas a dos plugins distintos dependiendo de su naturaleza, basándose estrictamente en una **convención de nombres** en los archivos de código:

- **`maven-surefire-plugin` (Pruebas Unitarias):**
    
    - Se ejecuta de forma nativa durante la fase `test` de Maven.
        
    - Escanea y ejecuta automáticamente los archivos que sigan los patrones: `/Test*.java`, `/*Test.java` o `/*TestCase.java`.
        
- **`maven-failsafe-plugin` (Pruebas de Integración):**
    
    - Se ejecuta durante las fases `pre-integration-test`, `integration-test` y `post-integration-test`.
        
    - Está diseñado para no hacer fallar el build inmediatamente si un componente se cae, permitiendo que las fases de limpieza se ejecuten.
        
    - Escanea los archivos con patrones: `/IT*.java`, `/*IT.java` o `/*ITCase.java`.

### 2. Configuración de Cobertura de Código con JaCoCo

**JaCoCo (Java Code Coverage)** es la herramienta estándar para medir qué porcentaje de las líneas de código han sido ejecutadas por la suite de pruebas. Se configura dentro del bloque `<plugins>` de tu `pom.xml`:

```
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.11</version>
    <executions>
        <execution>
            <id>prepare-agent</id>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>test</phase>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

Al ejecutar el comando `mvn clean test`, JaCoCo interceptará las pruebas de Surefire y volcará un reporte detallado en formato HTML, XML y CSV dentro de la ruta local `target/site/jacoco/index.html`.

## Guía 2: Configuración de Simulacros (Mocking) con Mockito

En las pruebas unitarias, es imperativo aislar la lógica del método bajo prueba eliminando las dependencias externas reales (como bases de datos, APIs de terceros o servicios complejos). Para ello se utiliza **Mockito**.

### Esquema de Configuración y Código Base

Para habilitar el framework de Mockito sin configuraciones manuales de ciclo de vida, se utiliza la anotación de extensión sobre la clase de pruebas:

```
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;
import static org.mockito.Mockito.*;
import static org.junit.jupiter.api.Assertions.*;

@ExtendWith(MockitoExtension.class)
public class UserServiceTest {

    // 1. Crear un sustituto virtual (Mock) de la dependencia externa
    @Mock
    private UserRepository userRepository;

    // 2. Inyectar automáticamente los mocks declarados dentro del servicio real bajo prueba
    @InjectMocks
    private UserService userService;

    @Test
    public void testGetPlayerSecureData() {
        // 3. Stubbing: Definir el comportamiento programado del Mock
        User mockUser = new User("javier", "role_admin");
        when(userRepository.findByUsername("javier")).thenReturn(mockUser);

        // 4. Ejecución del método a probar
        User result = userService.loadUserByUsername("javier");

        // 5. Aserción y Verificación
        assertNotNull(result);
        assertEquals("role_admin", result.getRole());
        
        // Verifica con precisión que el servicio invocó exactamente 1 vez al repositorio
        verify(userRepository, times(1)).findByUsername("javier");
    }
}
```

## Guía 3: Estrategia de Datos Simulados (Mock Data)

Cuando se ejecutan pruebas automatizadas de volumen o se requiere validar el comportamiento de interfaces frente a colecciones de datos masivas, escribir registros manualmente (`new User(...)`) es ineficiente y propenso a errores. El documento destaca tres soluciones automatizadas de la industria para generar sets de datos realistas y escalables bajo condiciones similares a las de producción:

1. **Faker.js:** Librería de JavaScript ampliamente utilizada para generar strings y metadatos aleatorios coherentes (nombres reales, direcciones, correos, números telefónicos) en lugar de usar textos aleatorios sin sentido (`"asdfasdf"`).
    
2. **Mockaroo:** Servicio en línea interactivo que permite diseñar plantillas complejas de bases de datos relacionales y exportar colecciones masivas de registros parametrizados en formatos CSV, JSON o SQL.
    
3. **JSON Generator:** Herramienta simplificada basada en plantillas de código para estructurar esquemas de datos JSON anidados dinámicos.