# 1. Planteamiento del problema y cambio de paradigma

La optimización logística clásica se formula, en general, como un problema combinatorio determinista: dada una red fija, con costos y restricciones conocidas, se busca una solución óptima **x*** que minimice una función objetivo escalar (por ejemplo, distancia, tiempo o costo monetario). Este enfoque es la base de modelos estándar como el camino mínimo, el problema del viajante (TSP) y los problemas de ruteo de vehículos (VRP).

Este paradigma, sin embargo, descansa sobre una suposición implícita fuerte: que la red subyacente es estructuralmente estable. En sistemas logísticos reales, dicha suposición rara vez se cumple. Las redes de transporte están expuestas de forma continua a disrupciones como fallos mecánicos, huelgas, eventos climáticos, restricciones regulatorias y efectos de cascada por congestión. Estas perturbaciones introducen incertidumbre no solo en los pesos de las aristas, sino en la existencia y capacidad efectiva de las propias conexiones.

En consecuencia, el problema real no consiste en encontrar una única ruta óptima bajo condiciones nominales, sino en seleccionar estrategias que permanezcan viables a lo largo de múltiples futuros posibles. Esto desplaza el foco desde la optimización determinista hacia la toma de decisiones bajo incertidumbre estructural, donde la topología de la red deja de ser un dato fijo y pasa a ser una variable aleatoria.

Prime Logistics adopta explícitamente este cambio de paradigma. En lugar de modelar la incertidumbre como ruido aditivo sobre costos o tiempos, el sistema la trata como riesgo epistemológico sobre la integridad de la red, capturado mediante perturbaciones estocásticas y posterior inferencia probabilística. El objetivo no es calcular una solución óptima global para un único mundo realizado, sino evaluar estrategias sobre un conjunto amplio de estados de red simulados y seleccionar aquellas que exhiben mayor robustez, redundancia y una distribución favorable del riesgo.

Formalmente, el problema no se reduce slamente a minimizar una función de costo determinista, sino a operar sobre el espacio de rutas y políticas factibles, identificando soluciones que logran compromisos aceptables entre eficiencia, fragilidad y resiliencia estructural. En este sentido, Prime Logistics reformula la optimización logística como un problema de decisión estadística, donde el aprendizaje, la inferencia y la estructura de la información son tan relevantes como la optimalidad clásica basada en grafos.

## Nota sobre implementación algorítmica

Si bien el enfoque de Prime Logistics se aleja de la optimización determinista clásica, la implementación se apoya en algoritmos bien establecidos de la teoría de grafos y la optimización combinatoria, utilizados como bloques instrumentales dentro de un marco estocástico más amplio.

En particular:

- Para la generación de soluciones base y la evaluación de rutas factibles se emplean algoritmos de camino mínimo (variantes de Dijkstra sobre grafos dirigidos y ponderados).

- La exploración de alternativas estructurales se realiza mediante resoluciones repetidas del problema de ruteo sobre instancias de red perturbadas estocásticamente.

- La selección final se basa en el análisis de soluciones candidatas sobre la frontera de Pareto, integrando múltiples métricas de desempeño.

Estos algoritmos actúan como mecanismos de proyección que permiten mapear el estado probabilístico de la red en decisiones operativas concretas.


# Bloque I
## Definición formal de la red logística (Gemelo Digital)

Esta sección establece el objeto matemático fundamental sobre el cual opera todo el framework de Prime Logistics. El propósito del Bloque I es construir un gemelo digital estructuralmente válido y matemáticamente explícito de la red logística física.

Toda simulación estocástica e inferencia posterior se define estrictamente como operadores que actúan sobre este objeto.


### 1. La Red como Grafo Dirigido con Atributos

El sistema logístico se modela como un grafo dirigido:

**G = (V, E)**

Donde:

- **V = {v₁, ..., vₙ}** es el conjunto finito de nodos logísticos (plantas, depósitos, clientes), identificados por coordenadas geográficas validadas (φ, λ).
- **E ⊆ V × V** es el conjunto de arcos dirigidos que representan enlaces de transporte físicamente factibles y activos.

Cada arco **eᵢⱼ ∈ E** está caracterizado por un vector de atributos escalares proyectados:

**w⃗ᵢⱼ = [cᵢⱼ, tᵢⱼ, kᵢⱼ]ᵀ**

Donde:

- **cᵢⱼ ∈ ℝ⁺**: Costo monetario determinista normalizado (resultado de la proyección de CostProfile sobre la distancia geodésica).
- **tᵢⱼ ∈ ℝ⁺**: Tiempo de tránsito esperado en condiciones nominales (basado en SpeedProfile).
- **kᵢⱼ ∈ ℝ⁺**: Capacidad máxima de flujo del arco (restricción física de la vía, no del vehículo).

Estas magnitudes se tratan como parámetros exógenos y observados. En este bloque no se les asigna interpretación probabilística alguna.

### 2. Representación matricial multicapa

En lugar de operar sobre los objetos, el sistema proyecta **G** a un espacio vectorial mediante una representación matricial multicapa. Esta es la descripción canónica del estado base de la red.

Sea **n = |V|**. Se definen las siguientes estructuras algebraicas:

#### A. Matrices de topología y flujo (ℝⁿˣⁿ)

**Matriz de adyacencia (A):** Define la conectividad topológica pura. Aᵢⱼ = 1 si existe camino valido, 0 no hay camino valido (i,j) ∈ E

**Matriz de costos (C):** Contiene los costos operativos **cᵢⱼ**.

**Matriz de tiempos (T):** Contiene los tiempos de tránsito **tᵢⱼ**.

**Matriz de capacidades de arco (K):** Define el límite superior de flujo permitido en el arco (i,j) (ej. tonelaje máximo de un puente o vía).

#### B. Vector de estado nodal (ℝⁿ)

**Vector de demanda neta (d):** d ∈ ℝⁿ = [d₁, d₂, ..., dₙ]ᵀ

Donde **dᵢ** representa el balance de carga del nodo:

- **dᵢ > 0**: Nodo de Demanda (Cliente)
- **dᵢ < 0**: Nodo de Oferta (Depósito/Planta)
- **dᵢ = 0**: Nodo de Transbordo (Paso)

Todas las matrices comparten el mismo espacio de índices **V × V**, garantizando coherencia estructural para operaciones de álgebra lineal vectorizada.

### 3. Validez topológica y restricciones de factibilidad

Antes de habilitar cualquier proceso estocástico, el gemelo digital debe satisfacer condiciones estrictas de factibilidad, garantizadas por el módulo NetworkValidator.

Formalmente, se imponen las siguientes restricciones:

#### Consistencia estructural
Todas las matrices de atributos (C, T, K) deben respetar el patrón de dispersión (sparsity pattern) inducido por **A**.

Aᵢⱼ = 0 ⇒ Cᵢⱼ, Tᵢⱼ, Kᵢⱼ = ∅ (o valor nulo/infinito según contexto)

#### Admisibilidad física
cᵢⱼ > 0, tᵢⱼ > 0, kᵢⱼ ≥ 0 ∀(i,j) ∈ E

Esto impide la existencia de ciclos de costo negativo o tiempos de viaje instantáneos que violen la causalidad.

#### Conectividad funcional
El subgrafo inducido por los arcos activos debe garantizar caminos dirigidos desde los nodos de oferta (i | dᵢ < 0) hacia los nodos de demanda (j | dⱼ > 0).

### 4. Separación estricta entre estructura e incertidumbre

Un principio central de diseño en Prime Logistics es la separación rigurosa entre **Topología (Bloque I)** y **Riesgo (Bloque II)**.

**En el bloque I:**
- Ningún arco posee probabilidad de fallo.
- Los costos y tiempos son valores escalares fijos (esperanzas matemáticas nominales).
- No se modela comportamiento aleatorio.

La red se trata como un sistema físico determinista e inmutable. La incertidumbre se introduce únicamente más adelante como un operador de perturbación externo.

### 5. Proyección algorítmica (Nota de implementación)

La construcción del gemelo digital utiliza algoritmos de grafos clásicos y álgebra matricial (numpy/scipy) como mecanismos de proyección.

- El cálculo de atributos utiliza distancia geodésica (Haversine) vectorizada.
- Los perfiles de costos complejos (CostProfile) se linealizan a valores escalares para construir la matriz **C**.

Su rol es interrogar la estructura estática.

La inteligencia del sistema emerge únicamente cuando estas proyecciones deterministas son sometidas a estrés estocástico en los bloques posteriores.

### 6. Rol Funcional dentro del pipeline

El Bloque I entrega:

- Una representación matricial explícita (A, C, T, K, d).
- Un gemelo digital validado y libre de ambigüedad semántica.
- Un sustrato limpio sobre el cual pueden actuar la simulación de Monte Carlo y la inferencia Bayesiana.


# Bloque II —Simulación estocástica y propagación de Riesgo

## 1. Propósito del bloque

El Chaos Engine es el motor de inferencia estocástica de Prime Logistics. Su objetivo es someter al Gemelo Digital (construido en el Bloque I) a un proceso de estrés sistemático mediante simulación Monte Carlo.

A diferencia de los análisis de riesgo tradicionales que evalúan fallos aislados, este motor construye Escenarios (Sₖ): narrativas coherentes de degradación donde múltiples eventos (sistémicos, tácticos y operativos) interactúan, se amplifican mutuamente y deforman la topología y los atributos de la red simultáneamente.

## 2. Definición del Estado Mutado

Sea **𝒩₀ = (A₀, C₀, T₀, K₀)** el estado base determinista definido en el Bloque I.

Un escenario **k** genera una Instantánea Mutada 𝒩ₖ:
𝒩ₖ = Γ(𝒩₀, Ωₖ, Sₖ)
Donde:

- **Ωₖ**: Conjunto de eventos activos en el escenario k.
- **Sₖ**: Índice de Estrés Acumulado (Stress Index) del escenario.
- **Γ**: Operador de mutación matricial (NetworkActor).

El estado mutado no es binario, es una deformación continua del espacio vectorial de la red (costos inflados, capacidades reducidas y conexiones cortadas).

## 3. Taxonomía de Eventos y Manifiesto

El universo de riesgos se define en un manifiesto declarativo, estructurado jerárquicamente en tres niveles de impacto:

- **SYSTEMIC (Sistémico)**: Eventos de alcance nacional/regional (ej. Paro Nacional, Inundación). Afectan la integridad macroscópica de la red.
- **TACTICAL (Táctico)**: Eventos zonales o sectoriales (ej. Bloqueo de Ruta, Corte de Energía).
- **MICRO (Operativo)**: Fricción diaria (ej. Falla Mecánica, Congestión).

Cada evento **Eᵢ** se define como una tupla:
Eᵢ = ⟨ Code, P_base, Target, Effects, Conditioners ⟩


## 4. Mecánica de Cascada (Probabilidad Efectiva)

El Chaos Engine no asume independencia entre sucesos. Implementa un modelo de inferencia causal simplificada donde la ocurrencia de eventos "padres" amplifica la probabilidad de eventos "hijos".

La probabilidad efectiva de activación de un evento **Eⱼ** dado un conjunto de eventos activos **Ω**, se calcula como:

P_eff(Eⱼ | Ω) = min(1.0, P_base(Eⱼ) × ∏ᵢ∈Ω φᵢ→ⱼ)

Donde **φᵢ→ⱼ** es el multiplicador de impacto definido en los Conditioners del manifiesto. Esto permite modelar colapsos en cadena (ej. Inundación → Bloqueo de Ruta).

## 5. Dinámica de Intensidad 

El sistema introduce una variable de estado global **S** (Stress Index). Cada evento activo contribuye una carga de estrés **ωᵢ** al sistema:

S = ∑ᵢ∈Ω ωᵢ

El impacto final de un evento sobre las métricas de la red (C o T) no es fijo, sino que escala dinámicamente con el estrés sistémico mediante el "Intensity Evaluator":

μ_final = 1 + (μ_base - 1) · (1 + λ · S)

Donde:

- **μ_base**: Multiplicador nominal del evento (ej. "el costo sube 20%").
- **λ**: Coeficiente de sensibilidad global (α para Tiempos, γ para Costos).

Esto modela la no-linealidad del caos: un mismo incidente es más dañino en un sistema que ya está estresado.

## 6. Operadores de Mutación Matricial

El NetworkActor aplica los impactos directamente sobre las matrices dispersas para eficiencia computacional:

#### Corte Topológico:
Si la acción es*DISABLE sobre un conjunto de arcos **ℐ**:

Aᵤᵥ ← 0, Kᵤᵥ ← 0 ∀ (u,v) ∈ ℐ

(Implementado mediante `tolil()`/`tocsr()` para manipulación rápida de estructuras dispersas).

#### Degradación de Capacidad:
Kᵤᵥ ← Kᵤᵥ · β_cap

#### Inflación de Métricas (Costos/Tiempos):
Cᵤᵥ ← Cᵤᵥ · μ_final(γ, S)
Tᵤᵥ ← Tᵤᵥ · μ_final(α, S)

## 7. Algoritmo de Generación Monte Carlo

El proceso de generación de un escenario **k** sigue una ejecución secuencial estricta:

1. **Clonación**: Se genera una copia profunda del estado base **𝒩₀**.
2. **Propagación por Niveles**: Se iteran los eventos en orden topológico (SYSTEMIC → TACTICAL → MICRO).
3. **Activación Estocástica**: Se evalúa **r ∼ U[0,1]**. Si **r < P_eff**, el evento se activa.
4. **Acumulación de Estrés**: Se actualiza **S ← S + weight(E)**.
5. **Mutación**: El NetworkActor deforma las matrices de **𝒩ₖ** según los efectos del evento y el **S** actual.

## 8. Criterio de Convergencia Estadística

Para evitar el sobrecómputo, el motor (MonteCarloEngine) monitorea la estabilidad estadística de la simulación en ventanas deslizantes (ej. 100 iteraciones).

La simulación se detiene anticipadamente si se satisfacen simultáneamente las condiciones de estabilidad en Media y Varianza para la métrica de reducción de capacidad:

| μ_window - μ_prev | < ε ∧ | σ²_window - σ²_prev | < ε

## 9. Salida del Bloque

El resultado es un objeto serializado (Pickle/JSON) que contiene:

- **El Conjunto de Escenarios**: **𝓢 = { Scenario₁, ..., Scenario_N }**.
- **Metadatos de Trazabilidad**: Qué eventos se activaron, sus probabilidades efectivas y el índice **S** resultante.
- **Estadísticas Agregadas**: Distribución de pérdidas de capacidad y frecuencias de eventos críticos.

Este conjunto **𝓢** constituye la entrada para el bloque III (Optimizador), que ya no optimizará sobre una red, sino sobre **N** redes mutadas.



# Bloque III — Inferencia de Riesgo Estructural

## 1. Propósito del Bloque

El Bayesian Auditor actúa como el tribunal forense del sistema. Su función es procesar la evidencia empírica generada por el bloque II para transformar "datos de simulación" en "conocimiento de confiabilidad".

A diferencia de un simple contador de fallas, este bloque implementa un motor de inferencia Bayesiana que:

- Audita cada escenario contra criterios de éxito/fracaso industrial.
- Actualiza la creencia sobre la confiabilidad de cada componente (Nodos y Arcos).
- Pondera la probabilidad de fallo con la severidad del impacto observado.

El resultado no es una estadística descriptiva, sino una matriz de riesgo predictiva (Φ) que guía al optimizador.

## 2. Auditoría Forense de Escenarios

El primer paso es determinista. El módulo **Auditor** somete cada escenario simulado **Sₖ** a un juicio binario basado en FailureCriteria.

Un escenario se declara **EXITOSO (Yₖ = 1)** si y solo si cumple simultáneamente:

1. **Integridad de Capacidad**: K_retained ≥ 85%
2. **Estabilidad de Tiempos**: T_travel ≤ 1.4 × T_base
3. **Eficiencia de Costos**: C_total ≤ 1.5 × C_base

Si alguna métrica viola el umbral, el escenario se marca como **FALLIDO (Yₖ = 0)** y se registran los componentes causales (Nodos/Arcos) identificados por el Bloque II.

## 3. Modelo de Inferencia Beta-Binomial

Para inferir la confiabilidad latente **θᵤ** de cada componente **u** (donde **u ∈ V ∪ E**), utilizamos el modelo conjugado Beta-Binomial.

### A. Priors conjugados
Asumimos una creencia inicial sobre la confiabilidad **θᵤ** (probabilidad de éxito):

θᵤ ∼ Beta(α₀, β₀)

- **α₀**: Peso de la evidencia de éxito previa (Prior).
- **β₀**: Peso de la evidencia de fallo previa.

### B. Actualización Bayesiana 
Al observar **N** escenarios, acumulamos éxitos (**sᵤ**) y fallos (**fᵤ**) específicos para el componente **u**. La distribución posterior es analítica exacta:

θᵤ | Data ∼ Beta(α₀ + sᵤ, β₀ + fᵤ)

Esto permite calcular la confiabilidad esperada (E[θᵤ]) y la varianza epistémica (incertidumbre de la estimación) sin costo computacional numérico.

## 4. Métrica de Fragilidad Compuesta (Riesgo)

Incorporamos la dimensión de impacto.

No todos los fallos son iguales. El besianJudge calcula la fragilidad **Ψᵤ** como el producto de la probabilidad de fallo y la severidad promedio observada.

**Probabilidad de fallo posterior:**

P(Fᵤ) = 1 - E[θᵤ] = 1 - α_post / (α_post + β_post)


**Impacto promedio condicional (Īᵤ):** 

Es el daño medio al sistema (ej. % de capacidad perdida) observado en los escenarios donde el componente **u** falló.

**Puntaje de fragilidad (Ψᵤ):**

Ψᵤ = P(Fᵤ) × Īᵤ

Este puntaje permite rankear los componentes: un nodo que falla poco pero catastróficamente puede tener mayor **Ψ** que uno que falla seguido pero sin consecuencias.

## 5. Construcción de la Matriz de Riesgo (L_total)

El InferenceEngine sintetiza el conocimiento en una matriz de riesgo de primer orden de tamaño **n × n**.

Cada celda **(i, j)** representa el riesgo combinado de intentar un transporte directo entre el nodo **i** y el nodo **j**.

El riesgo de la conexión **ℛᵢ→ⱼ** se modela como la probabilidad de fallo de la cadena lógica {Origen → Arco → Destino}, asumiendo independencia condicional en los fallos:

ℛᵢ→ⱼ = 1 - [(1 - Ψᵢ) × (1 - Ψⱼ) × (1 - Ψ_arc)]


Donde:
- **Ψᵢ**: Fragilidad del nodo origen.
- **Ψⱼ**: Fragilidad del nodo destino.
- **Ψ_arc**: Fragilidad del arco que conecta i → j.

## 6. Salida: Reliability Report

El bloque emite un objeto ReliabilityReport inmutable que contiene:

- **Risk matrix (ℛ)**: Tensor de riesgo para penalizar la función objetivo del optimizador.
- **Ranking de fragilidad**: Lista ordenada de los componentes más críticos.
- **Intervalos de Confianza**: Metadatos de varianza (σ²) para cada estimación, permitiendo distinguir entre "riesgo conocido" y "incertidumbre por falta de datos".

Este reporte constituye el mapa de navegación de riesgo que utilizará el bloque IV para tomar decisiones robustas.



# Bloque IV — Optimización estratégica y navegación de riesgo

## 1. Cambio de paradigma: del Grafo físico al grafo de decisión

El Prime Strategist opera sobre el grafo de riesgo Aumentado generado por el bloque III.

En este espacio vectorial, el "costo" de un arco es una composición vectorial de eficiencia operativa y costo de seguridad latente.

El objetivo del bloque es resolver un problema de decisión bajo incertidumbre estructural:

> ¿Cuánto está dispuesto a pagar el operador por reducir la varianza de su operación?

## 2. Scalarización paramétrica del riesgo (κ)

Para permitir el uso de motores de búsqueda deterministas de alto rendimiento (como el **DijkstraEngine** implementado), el sistema utiliza una técnica de **scalarización paramétrica**.

Se define el **peso generalizado (Wᵢⱼ)** de un arco como una función lineal del coeficiente de aversión al riesgo:

Wᵢⱼ(κ) = Cᵢⱼ + κ · ℛᵢⱼ

Donde:

- **Cᵢⱼ**: Costo monetario/temporal base (determinista).
- **κ (Kappa)**: Coeficiente de Aversión al Riesgo (el "precio sombra" que el usuario asigna a la seguridad).
- **ℛᵢⱼ**: Peso de Riesgo transformado.

Al variar **κ** desde **0** (neutralidad al riesgo) hasta **κₘₐₓ** (aversión total), el motor barre el espacio de soluciones y genera un conjunto de rutas candidatas óptimas para diferentes perfiles de decisión.

## 3. Transformación Isomorfa Logarítmica

Dado que la probabilidad de supervivencia de una ruta es multiplicativa (**P_ruta = ∏ pᵢ**), pero los algoritmos de grafos estándar operan sobre pesos aditivos, el motor aplica una transformación al espacio logarítmico.

La fragilidad **φᵢⱼ** (inferida en el bloque III) se transforma en un **peso de riesgo aditivo (ℛᵢⱼ)**:

ℛᵢⱼ = -ln(1 - φᵢⱼ)


**Nota de implementación:** Se utiliza la función `np.log1p(-phi)` para garantizar estabilidad numérica en valores de probabilidad cercanos a 0.

Esto asegura que minimizar la suma de **ℛᵢⱼ** sea matemáticamente equivalente a maximizar la probabilidad conjunta de supervivencia de la ruta.

## 4. Perfilado Estructural (Route Profiler)

Una vez hallada una ruta candidata, el sistema ejecuta una "biopsia estructural" (profiler.py) para calcular métricas de segundo orden que caracterizan la calidad del riesgo:

### A. Entropía Relativa (Incertidumbre de Shannon)
Mide la distribución del riesgo a lo largo de la ruta.

H_rel(R) = [-∑ pᵢ log₂ pᵢ] / log₂ |R|

- **H baja (< 0.3)**: Riesgo concentrado en un "punto unico de fallo". Estructuralmente frágil 
- **H alta (> 0.7)**: Riesgo distribuido uniformemente. Estructuralmente robusto por ausencia de eslabones críticos.

### B. Índice de Rigidez
Una métrica compuesta que evalúa la vulnerabilidad de la solución ante fallos catastróficos, combinando:
- **Impacto del arco crítico**: El riesgo del eslabón más débil.
- **Exposición nodal**: Porcentaje de nodos únicos visitados.
- **Volatilidad de costos**: Desviación estándar de los pesos en la ruta.
- **Vulnerabilidad de redundancia**: Ausencia de caminos alternativos (1 - RI).

## 5. Frontera de Pareto

El *Selector* evalúa las rutas candidatas generadas y construye la **Frontera de Pareto** en el espacio tridimensional:

( Minimizar Costo, Minimizar Rigidez, Maximizar Entropía )

El sistema descarta automáticamente cualquier solución Dominada (aquella para la cual existe otra opción que es mejor en todos los aspectos). Esto reduce el ruido decisional y presenta al usuario solo opciones eficientes.

## 6. Clasificación de arquetipos estratégicos

Para traducir la matemática compleja a un lenguaje de decisión humana, el *PrimeStrategicReporter* clasifica las soluciones supervivientes en *Arquetipos de Decisión*:

### El Unicornio
- **Perfil**: Bajo Costo / Alta Resiliencia.
- **Diagnóstico**: Una anomalía de mercado positiva. La opción dominante absoluta.

### El Tanque
- **Perfil**: Alto Costo / Máxima Resiliencia.
- **Diagnóstico**: Opción blindada para carga crítica. Enfoque "Confianza-minima".

### El Apostador
- **Perfil**: Mínimo Costo / Baja Resiliencia / Baja Entropía.
- **Diagnóstico**: Eficiente pero frágil. Depende de que un arco crítico específico no falle.

### El Equilibrista
- **Perfil**: Compromiso eficiente (Trade-off óptimo) según el **κ** actual.

## 7. Salida del Bloque: 

El Bloque IV entrega un reporte narrativo de inteligencia que incluye:

- **Auditoría Forense**: Validación de restricciones duras.
- **Navegacion profunda**: Desglose de Entropía, Rigidez e Índices de Redundancia.
- **Veredicto Táctico**: Una recomendación de acción clara ("Ejecutar", "Monitorear", "Descartar") y un nivel de confianza del sistema (recommendation_strength).

De esta forma, Prime Logistics trasciende la función de "calculadora de rutas" para convertirse en un **consultor estratégico automatizado**.

##  Limitaciones Conocidas y Suposiciones 

Para mantener la viabilidad computacional en este MVP, el modelo acepta los siguientes trade-offs teóricos:

### 1. **Independencia ingenua en la inferencia**
La actualización Beta-Binomial asume intercambiabilidad de las ejecuciones de simulación. Si bien se generan fallas correlacionadas (cascadas), el paso de inferencia trata la evidencia como pseudo-independiente para calcular la fragilidad local. Esto puede llevar a una confianza excesiva en el posterior para redes altamente acopladas.

### 2. **Enfoque Estructural vs. Operacional**
El modelo minimiza el *riesgo estructural* (disponibilidad de conexiones) en lugar de la *latencia operacional* (retrasos en colas). Actualmente no implementa dinámicas de colas M/G/k en los nodos; solo considera restricciones de capacidad puras.

### 3. **Flujo estático**
El optimizador actual asume enrutamiento estático por paso de simulación, ignorando las capacidades de re-enrutamiento dinámico de los agentes *durante* el evento de falla.


