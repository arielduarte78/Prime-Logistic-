# 1. Planteamiento del Problema y Cambio de Paradigma

La optimización logística clásica se formula, en general, como un problema combinatorio determinista: dada una red fija, con costos y restricciones conocidas, se busca una solución óptima **x*** que minimice una función objetivo escalar (por ejemplo, distancia, tiempo o costo monetario). Este enfoque es la base de modelos estándar como el Camino Mínimo, el Problema del Viajante (TSP) y los Problemas de Ruteo de Vehículos (VRP).

Este paradigma, sin embargo, descansa sobre una suposición implícita fuerte: que la red subyacente es estructuralmente estable. En sistemas logísticos reales, dicha suposición rara vez se cumple. Las redes de transporte están expuestas de forma continua a disrupciones como fallos mecánicos, huelgas, eventos climáticos, restricciones regulatorias y efectos de cascada por congestión. Estas perturbaciones introducen incertidumbre no solo en los pesos de las aristas, sino en la existencia y capacidad efectiva de las propias conexiones.

En consecuencia, el problema real no consiste en encontrar una única ruta óptima bajo condiciones nominales, sino en seleccionar estrategias que permanezcan viables a lo largo de múltiples futuros posibles. Esto desplaza el foco desde la optimización determinista hacia la toma de decisiones bajo incertidumbre estructural, donde la topología de la red deja de ser un dato fijo y pasa a ser una variable aleatoria.

Prime Logistics adopta explícitamente este cambio de paradigma. En lugar de modelar la incertidumbre como ruido aditivo sobre costos o tiempos, el sistema la trata como riesgo epistemológico sobre la integridad de la red, capturado mediante perturbaciones estocásticas y posterior inferencia probabilística. El objetivo no es calcular una solución óptima global para un único mundo realizado, sino evaluar estrategias sobre un conjunto amplio de estados de red simulados y seleccionar aquellas que exhiben mayor robustez, redundancia y una distribución favorable del riesgo.

Formalmente, el problema no se reduce a minimizar una función de costo determinista, sino a operar sobre el espacio de rutas y políticas factibles, identificando soluciones que logran compromisos aceptables entre eficiencia, fragilidad y resiliencia estructural. En este sentido, Prime Logistics reformula la optimización logística como un problema de decisión estadística, donde el aprendizaje, la inferencia y la estructura de la información son tan relevantes como la optimalidad clásica basada en grafos.

## Nota sobre Implementación Algorítmica

Si bien el enfoque de Prime Logistics se aleja de la optimización determinista clásica, la implementación se apoya en algoritmos bien establecidos de la teoría de grafos y la optimización combinatoria, utilizados como bloques instrumentales dentro de un marco estocástico más amplio.

En particular:

- **Para la generación de soluciones base y la evaluación de rutas factibles** se emplean algoritmos de camino mínimo (por ejemplo, variantes de Dijkstra sobre grafos dirigidos y ponderados).

- **La exploración de alternativas estructurales** se realiza mediante resoluciones repetidas del problema de ruteo sobre instancias de red perturbadas estocásticamente.

- **La selección final** no se basa en un único óptimo, sino en el análisis de soluciones candidatas sobre la frontera de Pareto, integrando múltiples métricas de desempeño.

Estos algoritmos no constituyen el núcleo conceptual del sistema, sino que actúan como mecanismos de proyección que permiten mapear el estado probabilístico de la red en decisiones operativas concretas.



#### Matrices de Atributos
- **Matriz de Costos C = (cᵢⱼ)**
- **Matriz de Tiempos T = (tᵢⱼ)**
- **Matriz de Capacidades K = (kᵢⱼ)**
- **Matriz de Distancias D = (dᵢⱼ)**

Todas las matrices comparten el mismo espacio de índices **V × V**, lo que garantiza coherencia estructural entre dimensiones.

Esta descomposición permite:

- Manipulación independiente de atributos físicos,
- Degradación controlada de dimensiones específicas,
- Preservación de invariantes topológicos durante perturbaciones estocásticas.

**La red no colapsa bajo estrés; transita por estados matemáticamente admisibles.**

### 3. Validez Topológica y Restricciones de Factibilidad

Antes de habilitar cualquier proceso estocástico o inferencial, el gemelo digital debe satisfacer condiciones mínimas de factibilidad.

Formalmente, se imponen las siguientes restricciones:

#### Consistencia Estructural
Todas las matrices respetan el patrón de dispersión inducido por **A**.

#### Admisibilidad Física
**cᵢⱼ > 0, tᵢⱼ > 0, kᵢⱼ > 0 ∀(i,j) ∈ E**

#### Conectividad Funcional
Para pares origen–destino designados **(s,t)**, existe al menos un camino dirigido en **G**.

Si alguna de estas condiciones se viola, la instancia es explícitamente rechazada. En ese caso, el problema no es de optimización bajo incertidumbre, sino de fallo en el diseño de la red.

### 4. Separación Estricta entre Estructura e Incertidumbre

Un principio central de diseño en Prime Logistics es la separación rigurosa entre topología y riesgo.

En el Bloque I:
- Ningún arco posee probabilidad de fallo,
- No se modela comportamiento aleatorio alguno,
- No se asume incertidumbre.

La red se trata como un sistema físico determinista.

La incertidumbre se introduce únicamente más adelante, como un operador externo que actúa sobre esta estructura.

Esta separación evita errores conceptuales frecuentes en modelos tradicionales, donde las suposiciones probabilísticas se incrustan prematuramente en la red.

### 5. Proyección Algorítmica (Nota de Implementación)

Si bien el framework es inherentemente no determinista, la construcción y validación del gemelo digital utilizan algoritmos clásicos de grafos como herramientas auxiliares de proyección.

En particular:

- Algoritmos de caminos mínimos (por ejemplo, Dijkstra) se emplean para verificar conectividad y generar rutas factibles de referencia,
- Dichos algoritmos operan exclusivamente sobre instancias deterministas de la red.

Su rol es interrogar la estructura, no tomar decisiones.

La inteligencia del sistema emerge únicamente cuando estas proyecciones deterministas son sometidas a estrés estocástico en los bloques posteriores.

### 6. Rol Funcional dentro del Pipeline

El Bloque I establece:

- Una representación matemática explícita de la red logística,
- Un gemelo digital validado y libre de ambigüedad semántica,
- Un sustrato limpio sobre el cual pueden actuar la simulación estocástica y la inferencia bayesiana.




Este estado determina la capacidad efectiva, conectividad y costos realizables de la red bajo dicho escenario.

## 3. Modelización de Sucesos Adversos Heterogéneos

El sistema define un conjunto finito de clases de sucesos adversos:

**S = {S₁, S₂, …, Sₘ}**

Cada clase representa un tipo cualitativamente distinto de perturbación logística (fallas mecánicas, eventos climáticos, conflictos laborales, shocks regulatorios, etc.).

A cada clase **Sℓ** se le asigna una variable aleatoria generadora:

**Zℓ ∼ Dℓ(θℓ)**

donde:

- **Dℓ** es una distribución adecuada a la naturaleza del evento,
- **θℓ** son parámetros de escala, frecuencia o severidad.

**Ejemplos típicos:**

- Eventos discretos raros → Poisson,
- Duración de interrupciones → Lognormal,
- Impactos extremos → distribuciones de cola pesada (Pareto),
- Fallas simples → Bernoulli.

Esto permite capturar la heterogeneidad estadística del riesgo, evitando reducir todo a una única probabilidad genérica.

## 4. Dependencias Condicionales entre Sucesos

El Chaos Engine no asume independencia entre sucesos. Por el contrario, admite relaciones de dependencia condicional entre eventos:

**P(Zⱼ | Zᵢ) ≠ P(Zⱼ)**

Estas dependencias representan fenómenos reales como:

- Fallas primarias que inducen fallas secundarias,
- Eventos sistémicos que amplifican vulnerabilidades locales,
- Shocks que se propagan espacial o funcionalmente.

Formalmente, el conjunto **S** puede representarse como un grafo dirigido acíclico parcial, análogo a una red bayesiana, donde:

- Los nodos representan clases de sucesos,
- Las aristas codifican relaciones de influencia causal.

## 5. Fallos Correlacionados en la Red

Dado un vector de sucesos **Z = (Z₁, …, Zₘ)**, el estado de cada arco deja de ser independiente.

La activación de un arco se modela como:

**Xᵢⱼ⁽ᵏ⁾ | φᵢⱼ, Z⁽ᵏ⁾ ∼ Bernoulli(1 − f(φᵢⱼ, Z⁽ᵏ⁾))**

donde:

- **φᵢⱼ** es la fragilidad latente del arco,
- **f(·)** es una función de impacto que incorpora efectos sistémicos.

Esto permite simular:

- Fallas simultáneas,
- Pérdida de capacidad correlacionada,
- Colapsos en cascada.

## 6. Generación de Escenarios Monte Carlo

Cada escenario **k** se genera mediante el siguiente proceso:

1. Muestreo de sucesos **Z⁽ᵏ⁾**.
2. Evaluación del impacto sobre cada arco.
3. Construcción del estado de red **X⁽ᵏ⁾**.
4. Cálculo de métricas agregadas (capacidad total, conectividad, costo realizable).

Este proceso se repite para **k = 1, …, N**, generando una distribución empírica del comportamiento del sistema.

## 7. Convergencia Estocástica Adaptativa

El número de escenarios **N** no es fijo. El Chaos Engine implementa un criterio de parada basado en estabilidad estadística.
Sea **K⁽ᵏ⁾** una métrica agregada (por ejemplo, capacidad total de la red). La simulación se detiene cuando:

**Δσᴋ² / σᴋ² < ε**

donde:

- **σᴋ²** es la varianza estimada,
- **Δσᴋ²** es el cambio entre lotes consecutivos,
- **ε** es un umbral de tolerancia.

Esto garantiza representatividad estadística sin sobrecómputo.

## 8. Salida del Bloque

El Chaos Engine produce un conjunto:

**X = {X⁽¹⁾, …, X⁽ᴺ⁾}**

junto con métricas asociadas por escenario.
Este conjunto constituye la evidencia empírica que alimenta al Bloque III.

## 9. Interpretación Fundamental

Este bloque modela el espacio de estrés plausible al que una red puede verse sometida.
La fragilidad no se asume, emerge.


# Bloque III — Bayesian Auditor: Inferencia de Fragilidad Bajo Evidencia No-IID

## 1. Rol del Bloque

El Bayesian Auditor transforma el conjunto de escenarios simulados por el Chaos Engine en conocimiento probabilístico estructurado.

Su función es inferir parámetros latentes del sistema: fragilidad, riesgo residual y confiabilidad efectiva de nodos y arcos, bajo condiciones de dependencia, correlación y evidencia sintética.

**Este bloque responde a una pregunta central:**

*Dado un conjunto de futuros posibles generados bajo estrés, ¿qué tan frágil es realmente cada componente de la red?*

## 2. Naturaleza del Problema Inferencial

Los datos producidos por el Bloque II no cumplen los supuestos clásicos de la inferencia estadística estándar:

- **No son independientes** (fallos correlacionados).
- **No son idénticamente distribuidos** (escenarios heterogéneos).
- **No provienen del mundo real**, sino de simulación estructurada.

Por lo tanto, el objetivo es actualizar creencias racionales sobre el comportamiento del sistema bajo estrés.

**Esto sitúa naturalmente el problema en un marco Bayesiano.**

## 3. Variable Latente de Fragilidad

Para cada componente estructural **u ∈ V ∪ E**, se define una variable latente:

**φᵤ ∈ [0, 1]**

donde **φᵤ** representa la probabilidad efectiva de fallo del componente bajo condiciones adversas.

Esta cantidad:

- No es observada directamente,
- No es constante en el tiempo,
- Resume tanto fragilidad intrínseca como exposición sistémica.

## 4. Modelo de Observación

A partir del Chaos Engine se obtiene, para cada componente **u**, una secuencia de observaciones binarias:

Estas observaciones no se asumen independientes, pero se utilizan como evidencia agregada.

Sea:

- **sᵤ = ∑ₖ Yᵤ⁽ᵏ⁾** (supervivencias),
- **fᵤ = N − sᵤ** (fallos).

## 5. Priors Informados (Creencias Iniciales)

Antes de observar la evidencia simulada, el sistema asume una creencia inicial sobre la confiabilidad del componente.

Se define:

**θᵤ = 1 − φᵤ**

y se asigna un prior Beta:

**θᵤ ∼ Beta(α₀, β₀)**

Este prior cumple dos funciones fundamentales:

1. Regulariza la inferencia ante pocos datos.
2. Codifica conocimiento previo (ingenieril o estratégico).

**Ejemplos:**

- Prior optimista → infraestructura robusta,
- Prior conservador → red frágil o desconocida.

## 6. Actualización Bayesiana (Conjugación)

Dado el carácter binario de las observaciones, se utiliza el modelo Beta-Binomial conjugado.

La distribución posterior queda definida como:

**θᵤ | Data ∼ Beta(α₀ + sᵤ, β₀ + fᵤ)**

Esta actualización es computacionalmente estable, interpretable y escalable.

## 7. Estimador de Fragilidad

La fragilidad inferida se define como la esperanza posterior:

**φ̂ᵤ = 1 − E[θᵤ] = 1 − (α₀ + sᵤ) / (α₀ + β₀ + N)**

Este valor es una medida racional de riesgo estructural, condicionada al espacio de estrés explorado.

## 8. Evidencia Correlacionada y Justificación Epistémica

Aunque las observaciones no son i.i.d., el uso del modelo Beta es epistémicamente válido porque:

- El objetivo es la evaluación de fragilidad.
- La correlación es inducida deliberadamente para revelar vulnerabilidades sistémicas.
- El modelo actúa como un agregador de evidencia bajo estrés.

En este contexto, el posterior refleja:

*"Qué tan frágil parece el componente, dado que fue sometido a futuros adversos plausibles".*

## 9. Construcción del Mapa de Fragilidad

El resultado del bloque es un vector:

**Φ = {φ̂ᵤ : u ∈ V ∪ E}**

Este mapa de fragilidad se convierte en una nueva capa semántica de la red, desacoplada de la topología física.

A partir de este punto:

- La red deja de ser solo geométrica,
- Pasa a ser probabilística y estratégica.

## 10. Salida del Bloque

El Bayesian Auditor produce:

- Fragilidad esperada por componente,
- Intervalos implícitos de incertidumbre,
- Métricas agregadas de riesgo estructural.

Este output alimenta directamente al Bloque IV, donde la optimización deja de ser determinista y pasa a ser una negociación explícita entre costo, riesgo y robustez.




## 4. Agregación del Riesgo

El riesgo total de una ruta no se modela como una suma de probabilidades.

Sea **φ̂ₑ** la fragilidad inferida de cada arco **e ∈ R**.

El sistema utiliza una agregación coherente con fallos en serie:

**Φ(R) = 1 − ∏ₑ∈ᴿ (1 − φ̂ₑ)**

Esto captura el hecho fundamental de que **una ruta falla si falla cualquiera de sus componentes críticos.**

## 5. Entropía como Medida de Fragilidad Estructural

Dos rutas pueden tener el mismo riesgo total y ser estructuralmente distintas.

Para capturar esta diferencia, se introduce la **Entropía de Shannon** sobre la distribución de riesgo:

**H(R) = −∑ₑ∈ᴿ pₑ log pₑ**  
con **pₑ = φ̂ₑ / ∑ₑ′∈ᴿ φ̂ₑ′**

La entropía mide cómo se distribuye el riesgo:

- **Baja entropía** → riesgo concentrado (Single Point of Failure),
- **Alta entropía** → riesgo distribuido.

Se utiliza una versión normalizada:

**Hₙₒᵣₘ(R) = H(R) / log |R|**

## 6. Optimización Multiobjetivo

El sistema **no colapsa** los objetivos en una única métrica arbitraria.

En su lugar, construye la **Frontera de Pareto** en el espacio:

**(C(R), Φ(R), 1 − Hₙₒᵣₘ(R))**

Una ruta es Pareto-dominada si existe otra que:

- No es peor en ningún criterio,
- Y es estrictamente mejor en al menos uno.

El resultado es un conjunto de soluciones eficientes.

## 7. Escalarización Estratégica (Opcional)

Cuando se requiere una decisión puntual, se utiliza una función escalarizada:

**Z(R) = C(R) + λ · Φ(R) + γ · (1 − Hₙₒᵣₘ(R))**

donde:

- **λ**: aversión al riesgo,
- **γ**: penalización por rigidez estructural.

Estos parámetros son estratégicos. 
Representan la postura de decisión del operador.

## 8. Algoritmos Utilizados

Para exploración y resolución:

- Variantes de Dijkstra multi-criterio,
- Poda por dominancia de Pareto,
- Heurísticas de enumeración de rutas factibles.
Se busca la insuficiencia decisional bajo restricciones reales.

## 9. Clasificación Estratégica de Soluciones

Las rutas resultantes se agrupan en arquetipos interpretables:

- **El Unicornio**: bajo costo, bajo riesgo (raro).
- **El Tanque**: alto costo, extrema robustez.
- **El Equilibrista**: compromiso eficiente.
- **El Ilusionista**: barato pero frágil (riesgo oculto).

Esto convierte el output matemático en lenguaje de decisión humana.

## 10. Interpretación Final

Este bloque formaliza una idea central:

> En sistemas complejos, decidir es elegir qué error estás dispuesto a tolerar.

Prime Logistics no promete certezas.  
Entrega algo más valioso:

- Visibilidad de trade-offs,
- Cuantificación del riesgo,
- Y decisiones que siguen siendo válidas cuando el mundo no coopera.



## 11. Limitaciones Conocidas y Suposiciones 

Para mantener la viabilidad computacional en este MVP, el modelo acepta los siguientes trade-offs teóricos:

### 1. **Independencia Ingenua en la Inferencia**
La actualización Beta-Binomial asume intercambiabilidad de las ejecuciones de simulación. Si bien el Chaos Engine genera fallas correlacionadas (cascadas), el paso de inferencia trata la evidencia como pseudo-independiente para calcular la fragilidad local. Esto puede llevar a una confianza excesiva en el posterior para redes altamente acopladas.

### 2. **Enfoque Estructural vs. Operacional**
El modelo minimiza el riesgo estructural (disponibilidad de conexiones) en lugar de la latencia operacional (retrasos en colas). Actualmente no implementa dinámicas de colas M/G/k en los nodos; solo considera restricciones de capacidad puras.

### 3. **Flujo Estático**
El optimizador actual asume enrutamiento estático por paso de simulación, ignorando las capacidades de re-enrutamiento dinámico de los agentes durante el evento de falla.

**Estas restricciones son decisiones de diseño deliberadas para priorizar la escala (>10k nodos) sobre la precisión de micro-simulación.**

