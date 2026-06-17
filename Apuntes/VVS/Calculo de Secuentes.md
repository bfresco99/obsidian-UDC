El cálculo de secuentes trabaja con expresiones de la forma Γ⊢Δ, donde Γ es el **antecedente** (un conjunto de premisas asumidas como verdaderas) y Δ es el **sucesor** o consecuente (una disyunción de posibles conclusiones). El objetivo es demostrar que el secuente es un **axioma** (cuando una misma fórmula A aparece a ambos lados: Γ,A⊢A,Δ).

Para resolverlo de forma eficiente, no trabajes de arriba a abajo (construyendo la deducción), sino **de abajo a arriba** (descomponiendo tu objetivo hasta llegar a axiomas).

[[representacion_reglas_de_secuentes.png]]

### 1. Clasificación Estratégica de las Reglas

El truco para no atascarse es saber qué regla aplicar primero. Las reglas lógicas se dividen según el conector principal y el lado del secuente donde se aplican (Izquierda o Derecha). Además, se clasifican en dos tipos fundamentales:

- **Reglas Invertibles (No pierden información):** Se pueden aplicar en cualquier momento porque, si el secuente original es válido, los secuentes resultantes también lo serán.
    
- **Reglas No Invertibles (Toman decisiones):** Al aplicarlas puedes "perder" información crucial. Debes retrasar su uso el mayor tiempo posible.

[[reglas_estructurales.png]]

[[reglas_negacion.png]]

[[reglas_disyuncion.png]]

[[reglas_conjuncion.png]]

[[reglas_implicacion.png]]

[[reglas_equivalencia.png]]

### 2. Algoritmo de Aplicación Paso a Paso

Sigue estrictamente este orden de prioridad en cada paso de tu árbol de derivación:

#### Paso 1: Aplicar Reglas Invertibles de una Premisa (Sin Ramificación)

Limpia el secuente reduciendo la complejidad de las fórmulas sin duplicar tu trabajo.

- **Negación a la Derecha (¬R):** Pasa la fórmula al antecedente quitando el negador.
    
    Γ,A⊢Δ
    ------------
	Γ⊢¬A,Δ​
    
- **Negación a la Izquierda (¬L):** Pasa la fórmula al sucesor quitando el negador.
    
    Γ⊢A,Δ​
    ------------
    Γ,¬A⊢Δ
    
- **Implicación a la Derecha (→R):** El antecedente de la implicación pasa a las premisas, el consecuente se queda a la derecha.
    
    Γ,A⊢B,Δ​
    -------------
    Γ⊢A→B,Δ
    
- **Conjunción a la Izquierda (∧L):** Separa las dos partes en el antecedente.
    
    Γ,A,B⊢Δ​
    -------------
    Γ,A∧B⊢Δ
    
- **Disyunción a la Derecha (∨R):** Separa las dos partes en el sucesor.
    
    Γ⊢A,B,Δ​
	-------------
	Γ⊢A∨B,Δ

#### Paso 2: Aplicar Reglas Invertibles de Dos Premisas (Con Ramificación)

Aplica estas reglas solo cuando hayas agotado el Paso 1. Tu árbol se dividirá en dos ramas independientes que tendrás que resolver por separado.

- **Conjunción a la Derecha (∧R):** Obliga a demostrar ambas partes por separado.
    
    Γ⊢A,Δ  Γ⊢B,Δ​
    --------------
    Γ⊢A∧B,Δ
    
- **Disyunción a la Izquierda (∨L):** Analiza por casos (si ocurre A o si ocurre B).
    
    Γ,A⊢Δ  Γ,B⊢Δ​
    ---------------
    Γ,A∨B⊢Δ
    
- **Implicación a la Izquierda (→L):** Genera dos ramas: una para demostrar el subobjetivo A y otra donde ya dispones de la conclusión B.
    
    Γ⊢A,Δ  Γ,B⊢Δ​
    ----------------
    Γ,A→B⊢Δ
#### Paso 3: Tratar con Cuantificadores (Lógica de Predicados)

Si aparecen cuantificadores, el orden es estricto para evitar violar restricciones de variables:

1. **Primero (∀R y ∃L):** Introducen una constante nueva (una variable fresca a que no haya aparecido antes en el secuente).
    
    Γ⊢A[x↦a],Δ                ​Γ,A[x↦a]⊢Δ​
    ---------------                  ---------------
    Γ⊢∀xA,Δ                     Γ,∃xA⊢Δ
    
2. **Último (∀L y ∃R):** Te permiten sustituir la variable por **cualquier término** t que te convenga para forzar un axioma. No eliminan la fórmula original por si necesitas usarla con otro término más adelante.
    
    Γ,∀xA,A[x↦t]⊢Δ
    ---------------
    Γ,∀xA⊢Δ
    
    Γ⊢∃xA,A[x↦t],Δ​
    ---------------
    Γ⊢∃xA,Δ

