En PVS, las premisas del antecedente se numeran con enteros negativos `[-1]`, `[-2]`, etc., y las fórmulas del sucesor con enteros positivos `[1]`, `[2]`, etc. El objetivo es reducir el secuente a vaciarse o encontrar contradicciones directas.

### 1. Tabla de Comandos Básicos y Equivalencias Lógicas

| **Operación Lógica**                                | **Equivalente en Cálculo de Secuentes**                       | **Comando PVS**                       | **Descripción**                                                                    |
| --------------------------------------------------- | ------------------------------------------------------------- | ------------------------------------- | ---------------------------------------------------------------------------------- |
| **Simplificación Básica**                           | Reglas de una premisa ($\land L$, $\lor R$, $\rightarrow R$)  | `(flatten)`                           | Descompone conectores que no ramifican el árbol de prueba.                         |
| **Ramificación**                                    | Reglas de dos premisas ($\land R$, $\lor L$, $\rightarrow L$) | `(split)`                             | Divide el secuente actual en subgoles independientes.                              |
| **Automatización Prop.**                            | Combina `flatten` y `split` iterativamente                    | `(prop)`                              | Resuelve o ramifica toda la lógica proposicional automáticamente.                  |
| **Cuantificador Universal Der. / Existencial Izq.** | $\forall R$ / $\exists L$ (Variable fresca)                   | `(skolem!)` o `(skolem número ("a"))` | Introduce constantes de Skolem para eliminar el cuantificador.                     |
| **Cuantificador Existencial Der. / Universal Izq.** | $\exists R$ / $\forall L$ (Sustitución por término)           | `(inst número "término")`             | Instancia la variable cuantificada con el término específico que necesitas.        |
| **Uso de Definiciones**                             | Expansión de funciones o lemas externos                       | `(expand "nombre")`                   | Reemplaza un identificador por su definición explícita.                            |
| **Uso de Lemas**                                    | Introducción de axiomas/teoremas al antecedente               | `(lemma "nombre")`                    | Añade el lema indicado como una nueva línea en los antecedentes negativos.         |
| **Sustitución por Igualdad**                        | Aplicación de un puente $x = y$                               | `(replace número_igualdad)`           | Usa una igualdad del secuente para sustituir los términos en el resto de fórmulas. |
| **Automatización Completa**                         | Prop + Skolem + Simplificación aritmética                     | `(grind)`                             | El comando "bomba". Intenta resolver todo el subgol de manera agresiva.            |
### 2. Flujo de Trabajo Típico en el Prover

A continuación se detalla el orden de comandos recomendado cuando te enfrentas a un secuente en blanco dentro del entorno interactivo de PVS:

**1. Simplificación inicial y Skolemización: Paso Obligatorio.**

Lanza `(skolem!)` seguido de `(flatten)`. Esto limpiará todos los cuantificadores universales del sucesor e introducirá constantes limpias antes de realizar cualquier sustitución manual.

**2. Expansión de definiciones del dominio: Contextual.**

Si tu secuente contiene funciones propias o de librería (ej. `f(x)`), utiliza `(expand "f")` para revelar la lógica matemática subyacente que necesitas manipular.

**3. Instanciación dirigida: Si quedan cuantificadores.**

Identifica las fórmulas con ∃ a la derecha o ∀ a la izquierda. Busca qué constante de las introducidas en el Paso 1 encaja en la propiedad y ejecútala usando `(inst número_fórmula "mi_constante")`.

**4. Resolución proposicional y aritmética: Cierre de ramas.**

Aplica `(prop)` o `(ground)`. Si el secuente está bien planteado y realizaste las instanciaciones correctas, este comando cerrará las ramas actuales mostrando el mensaje `This completes the proof of...`.

**5. Navegación entre subgoles: Solo si el paso anterior ramificó.**

Si `(split)` o `(prop)` generaron múltiples subramas, usa `(postpone)` para dejar el subgol actual de lado y saltar al siguiente si te encuentras encallado.