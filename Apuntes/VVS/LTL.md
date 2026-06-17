## 1. Sintaxis y Operadores de LTL

La Lógica Temporal Lineal extiende la lógica proposicional estándar sumando operadores modales que permiten razonar sobre una línea de tiempo unidireccional e infinita.

Partiendo de un conjunto de átomos o proposiciones Σ (ej. {p,q,r}) , LTL utiliza dos tipos de operadores modales:

### Operadores Unarios (Un argumento)

- □A: **"Forever"** / Siempre (el cuadrado o _G_ de _Globally_). Indica que A es verdadero ahora y en todos los estados futuros.
    
- ⋄A: **"Eventually"** / Alguna vez (el rombo o _F_ de _Finally_). Indica que A será verdadero ahora o en algún momento del futuro.
    
- ◯A: **"Next"** / Siguiente estado (el círculo o _X_ de _eXtend_). Indica que A es verdadero en el estado inmediatamente posterior.
    

### Operadores Binarios (Dos argumentos)

- A U B: **"Until"** / Hasta. Exige que B sea verdadero en algún momento futuro (obligatoriamente) y que A sea verdadero en todos los estados anteriores a ese punto.
    
- A W B: **"Weak Until"** / Hasta débil. Es igual que el _Until_, pero quita la obligación de que B ocurra. Si B nunca ocurre, A tiene que mantenerse verdadero para siempre.
    
- A R B: **"Release"** / Liberación. Es el dual de U. Significa que B se mantiene verdadero hasta el momento (inclusive) en que A se vuelve verdadero, liberándolo de la obligación. Si A nunca ocurre, B debe ser verdadero por siempre.
    

> **Prioridad de Operadores:** Los operadores modales (□,⋄,◯) y los binarios (U,R,W) tienen la máxima prioridad, seguidos en orden descendente por la conjunción (∧), la disyunción (∨) y finalmente la implicación (→) y coimplicación (↔).

## 2. Semántica: Trazas y Satisfacibilidad

- **Estado (s):** Es una valoración de verdad que asigna _Verdadero_ o _Falso_ a cada átomo. Se representa como el conjunto de átomos que son verdaderos en ese instante.
    
- **Interpretación o Traza (M):** Es una secuencia **infinita** de estados (s0​,s1​,s2​,…) que simula el transcurso del tiempo.
    

La relación de **satisfacibilidad** (M,i⊨α) evalúa si una fórmula α es verdadera en la traza M en un punto de tiempo específico i≥0:

- M,i⊨p⟺p∈si​.
    
- M,i⊨□α⟺∀j≥i, M,j⊨α.
    
- M,i⊨⋄α⟺∃j≥i tal que M,j⊨α.
    
- M,i⊨◯α⟺M,i+1⊨α.
    

### Terminología Estándar

- **Modelo:** Una traza M es modelo de una teoría si satisface todas sus fórmulas en el instante inicial 0 (M,0⊨α).
    
- **Tautología (Válida):** Cuando cualquier interpretación/traza posible es un modelo de la fórmula.
    
- **Insatisfacible (Inconsistente):** Cuando la fórmula no tiene ningún modelo posible.

## 3. Equivalencias Notables y Traducción de Kamp

LTL comparte una profunda relación con la **Lógica de Primer Orden Monádica con la relación menor que (MFO(<))**.

### Teorema de Kamp (1968)

LTL tiene exactamente la misma expresividad matemática que MFO(<). Cualquier fórmula temporal se puede mapear a una fórmula de predicados donde los argumentos representan puntos en el tiempo (i,j,k) y el operador +1 avanza un estado.

- (◯α)(i) =def α(i+1)
    
- (□α)(i) =def ∀j≥i:α(j)
    
- (⋄α)(i) =def ∃j≥i:α(j)
    
- (α U β)(i) =def ∃j≥i:(β(j)∧(∀k∈i…j−1:α(k)))
    

### Descomposiciones temporales fundamentales

Estas equivalencias permiten expandir el estado actual separándolo del futuro inmediato:

- □α↔α∧◯□α
    
- ⋄α↔α∨◯⋄α
    
- α U β↔β∨(α∧◯(α U β))

## 4. Especificación de Propiedades del Software

El uso principal de LTL es el _Model Checking_ (verificación de modelos) mediante la formalización de requisitos en tres grandes categorías:

1. **Propiedades de Seguridad (Safety):** Garantizan que **"nunca ocurra algo malo"** (□¬malo). Un contraejemplo de seguridad es **finito**: basta con mostrar la ejecución hasta el estado donde ocurrió el fallo.
    
2. **Propiedades de Vitalidad (Liveness):** Garantizan que **"algo bueno eventualmente sucederá"** (□(peticioˊn→⋄respuesta)). Un contraejemplo de vitalidad es **infinito**; no obstante, en LTL siempre se puede representar mediante un lazo cerrado periódico (un contraejemplo cíclico).
    
3. **Propiedades de Equidad (Fairness):** Aseguran que si el sistema tiene múltiples opciones, estas se seleccionen de forma justa.
    
    - _Equidad Fuerte:_ Si un proceso está habilitado infinitas veces, debe ejecutarse infinitas veces (□⋄habilitado→□⋄ejecutado).
        
    - _Equidad Débil:_ Si un proceso está habilitado continuamente (sin interrupción), debe ejecutarse infinitas veces (⋄□habilitado→□⋄ejecutado).

## 5. Complejidad y Expresividad Frente a Autómatas

- **Complejidad:** El problema de decidir si una fórmula LTL es satisfacible es **PSPACE-completo**. Esto significa que requiere una cantidad de memoria/espacio que escala de forma polinomial, situándose de manera sospechosa por encima de los problemas NP-completos.
    
- **LTL vs Autómatas de Büchi (BA):** En la práctica de model checking, las fórmulas LTL se convierten en Autómatas de Büchi equivalentes (autómatas de estados finitos que aceptan palabras de longitud infinita si visitan un estado final infinitas veces).
    
- **Limitación de Expresividad:** LTL es **menos expresivo** que los Autómatas de Büchi. No puede expresar eventos que dependan de estados alternos fijos; por ejemplo, la propiedad _"p es verdadero en todos los estados pares (s0​,s2​,s4​…) y libre en los impares"_ **no se puede escribir en LTL**, pero sí se puede modelar con un Autómata de Büchi.

