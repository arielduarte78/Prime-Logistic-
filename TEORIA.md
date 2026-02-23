# Prime Logistics: Optimización Logística bajo Incertidumbre Estructural

## 1. Planteamiento del problema y cambio de paradigma

La optimización logística clásica se formula, en general, como un problema combinatorio determinista: dada una red fija, con costos y restricciones conocidas, se busca una solución óptima \(x^*\) que minimice una función objetivo escalar (por ejemplo, distancia, tiempo o costo monetario). Este enfoque es la base de modelos estándar como el camino mínimo, el problema del viajante (TSP) y los problemas de ruteo de vehículos (VRP).

Este paradigma, sin embargo, descansa sobre una suposición implícita fuerte: que la red subyacente es **estructuralmente estable**. En sistemas logísticos reales, dicha suposición rara vez se cumple. Las redes de transporte están expuestas de forma continua a disrupciones como fallos mecánicos, huelgas, eventos climáticos, restricciones regulatorias y efectos de cascada por congestión. Estas perturbaciones introducen incertidumbre no solo en los pesos de las aristas, sino en la existencia y capacidad efectiva de las propias conexiones.

En consecuencia, el problema real no consiste en encontrar una única ruta óptima bajo condiciones nominales, sino en seleccionar estrategias que permanezcan viables a lo largo de múltiples futuros posibles. Esto desplaza el foco desde la optimización determinista hacia la **toma de decisiones bajo incertidumbre estructural**, donde la topología de la red deja de ser un dato fijo y pasa a ser una variable aleatoria.

Prime Logistics adopta explícitamente este cambio de paradigma. En lugar de modelar la incertidumbre como ruido aditivo sobre costos o tiempos, el sistema la trata como **riesgo epistemológico sobre la integridad de la red**, capturado mediante perturbaciones estocásticas y posterior inferencia probabilística. El objetivo no es calcular una solución óptima global para un único mundo realizado, sino evaluar estrategias sobre un conjunto amplio de estados de red simulados y seleccionar aquellas que exhiben mayor robustez, redundancia y una distribución favorable del riesgo.

Formalmente, el problema no se reduce solamente a minimizar una función de costo determinista, sino a operar sobre el espacio de rutas y políticas factibles, identificando soluciones que logran compromisos aceptables entre eficiencia, fragilidad y resiliencia estructural. En este sentido, Prime Logistics reformula la optimización logística como un **problema de decisión estadística**, donde el aprendizaje, la inferencia y la estructura de la información son tan relevantes como la optimalidad clásica basada en grafos.

### Nota sobre implementación algorítmica

Si bien el enfoque de Prime Logistics se aleja de la optimización determinista clásica, la implementación se apoya en algoritmos bien establecidos de la teoría de grafos y la optimización combinatoria, utilizados como bloques instrumentales dentro de un marco estocástico más amplio.

En particular:
- Para la generación de soluciones base y la evaluación de rutas factibles se emplean algoritmos de camino mínimo (variantes de Dijkstra sobre grafos dirigidos y ponderados).
- La exploración de alternativas estructurales se realiza mediante resoluciones repetidas del problema de ruteo sobre instancias de red perturbadas estocásticamente.
- La selección final se basa en el análisis de soluciones candidatas sobre la **frontera de Pareto exacta**, integrando múltiples métricas de desempeño mediante dominancia espacial.

Estos algoritmos actúan como mecanismos de proyección que permiten mapear el estado probabilístico de la red en decisiones operativas concretas.

---

## Bloque I — Ingesta Geoespacial y Topología Dinámica (V1.2)

La versión 1.2 erradica la dependencia de grafos estáticos precompilados. El Bloque I actúa como un motor geoespacial de instanciación bajo demanda (**Smart Lazy Loading**), transformando coordenadas y vectores de intención comercial en micro-grafos viales topológicamente válidos.

### 1. Resolución Topológica Bajo Demanda

El sistema implementa una arquitectura de memoria en dos niveles para evitar el colapso computacional frente a datos geoespaciales a escala país:
- **Pre-Warming (Nodos Críticos)**: Áreas de alta densidad logística (ej. centros de distribución primarios, AMBA) se mantienen en memoria precomputada.
- **Smart Lazy Loading**: Cuando se exige un corredor inédito, el sistema identifica el vacío espacial, descarga exclusivamente la porción del polígono terrestre requerida, la purga de nodos aislados y la fusiona con la red global. Consultas subsecuentes sobre el mismo corredor operan con latencia cero a través del sistema de caché.

### 2. Definición Nodal (Instalaciones Macro)

Un **nodo** en este paradigma no representa intersecciones viales, sino **instalaciones físicas** que inyectan o absorben masa del sistema.

| Campo          | Tipo    | Requisito | Descripción                                      |
|----------------|---------|-----------|--------------------------------------------------|
| id             | String  | Obligatorio | Identificador único (ej. NODE_0001).             |
| lat / lon      | Float   | Obligatorio | Coordenadas en grados decimales (WGS84).        |
| supply         | Float   | Opcional  | Capacidad de despacho en toneladas métricas.     |
| demand         | Float   | Opcional  | Necesidad de recepción en tonelétricas.          |
| station_type   | String  | Obligatorio | Categoría funcional (warehouse, factory, port, etc.). |
| proc_cost      | Float   | Opcional  | Costo fijo unitario por procesamiento en la instalación. |

**Balance de Masa Nodal**:
El comportamiento topológico del nodo se deduce determinísticamente mediante la ecuación de demanda neta:

\[
d_i = \text{Demand}_i - \text{Supply}_i
\]

- \(d_i < 0\): Nodo Fuente (Inyección de carga).
- \(d_i > 0\): Nodo Sumidero (Absorción de carga).
- \(d_i = 0\): Nodo de Transbordo estricto.

### 3. Definición de Arcos (Intenciones de Corredor)

Un **arco** ya no es un segmento de asfalto, sino un **vector de intención de viaje**. El motor geoespacial traduce esta intención en la infraestructura física subyacente.

| Campo          | Tipo    | Requisito | Descripción                                              |
|----------------|---------|-----------|----------------------------------------------------------|
| id             | String  | Obligatorio | Identificador único del corredor (ej. ARC_0010).         |
| origin         | String  | Obligatorio | ID del nodo de salida.                                   |
| destination    | String  | Obligatorio | ID del nodo de llegada.                                  |
| mode           | String  | Obligatorio | Perfil cinemático (truck, car, bus) que dicta la red a extraer. |
| vehicle_cap    | Float   | Opcional  | Restricción de carga máxima del transporte asignado.     |

### 4. Indexación Espacial (H3) y Jerarquía Topológica

Para evitar la descarga ineficiente de polígonos masivos, el sistema fragmenta el globo utilizando el índice espacial **H3** (diseñado por Uber).

1. **Definición del Pasillo**: Se proyecta una traza lineal entre el origen y el destino. El sistema recubre esta traza activando celdas H3, inyectando anillos concéntricos periféricos (buffers) para garantizar grados de libertad en el ruteo (desvíos por rutas secundarias).
2. **Filtro de Jerarquía Logística**: Se implementa un modelo de resolución asimétrica:
   - **Puntas (Terminales)**: En los hexágonos que contienen al Origen y Destino, se extrae la red vial completa (Nivel 1: calles locales, avenidas) para garantizar capilaridad en la última milla.
   - **Corredor Troncal**: En los hexágonos intermedios, el motor aplica un filtro estricto de Nivel 3 (motorways, trunks, primary roads), descartando infraestructura urbana irrelevante para carga pesada y colapsando drásticamente el tamaño matricial del problema.

### 5. Estándares, Fallbacks y Arquitectura Estructural

La confiabilidad del gemelo digital depende de su tolerancia a anomalías en las fuentes externas (OpenStreetMap):
- **Bóveda de Caché Criptográfica**: Todo subgrafo extraído se somete a hashing SHA-256 y escritura atómica (FileLock). Si un archivo GraphML resulta corrupto por interrupción de red, se aísla en cuarentena y se fuerza la re-descarga.
- **Geometría de Fallback**: Si la extracción vial falla por ausencia de datos en el servidor externo, el sistema degrada de manera segura hacia la distancia geodésica utilizando la métrica de Haversine (radio terrestre de 6371 km).
- **Traducción Matricial**: El módulo `topology.py` absorbe el grafo fusionado y lo proyecta en las matrices algebraicas definitivas de Adyacencia (\(A\)), Costo (\(C\)), Tiempo (\(T\)) y Capacidad (\(K\)), cerrando la brecha entre la geografía y el álgebra lineal requerida por el Bloque II.

---

## Bloque II — Simulación estocástica y propagación de Riesgo

### 1. Propósito del bloque

El **Chaos Engine** es el motor de inferencia estocástica de Prime Logistics. Su objetivo es someter al Gemelo Digital a un proceso de estrés sistemático mediante simulación Monte Carlo.

A diferencia de los análisis de riesgo tradicionales que evalúan fallos aislados, este motor construye **Escenarios** (\(S_k\)): narrativas coherentes de degradación donde múltiples eventos interactúan, se amplifican mutuamente y deforman la topología y los atributos de la red simultáneamente.

### 2. Definición del Estado Mutado

Sea \(\mathcal{N}_0 = (A_0, C_0, T_0, K_0)\) el estado base determinista.

Un escenario \(k\) genera una **Instantánea Mutada** \(\mathcal{N}_k\):

\[
\mathcal{N}_k = \Gamma(\mathcal{N}_0, \Omega_k, S_k)
\]

Donde:
- \(\Omega_k\): Conjunto de eventos activos.
- \(S_k\): Índice de Estrés Acumulado.
- \(\Gamma\): Operador de mutación matricial.

El estado mutado no es binario, es una deformación continua del espacio vectorial de la red.

### 3. Taxonomía de Eventos y Manifiesto

El universo de riesgos se define en un manifiesto declarativo:
- **SYSTEMIC (Sistémico)**: Eventos de alcance nacional/regional.
- **TACTICAL (Táctico)**: Eventos zonales o sectoriales.
- **MICRO (Operativo)**: Fricción diaria.

Cada evento \(E_i\) se define como:

\[
E_i = \langle Code, P_{base}, Target, Effects, Conditioners \rangle
\]

### 4. Mecánica de Cascada (Probabilidad Efectiva)

El Chaos Engine no asume independencia entre sucesos. Implementa un modelo de inferencia causal donde la ocurrencia de eventos "padres" amplifica la probabilidad de eventos "hijos".

\[
P_{eff}(E_j | \Omega) = \min\left(1.0, P_{base}(E_j) \times \prod_{i \in \Omega} \phi_{i \to j}\right)
\]

### 5. Dinámica de Intensidad

El sistema introduce una variable de estado global \(S\) (Stress Index).

\[
S = \sum_{i \in \Omega} \omega_i
\]

El impacto final de un evento sobre las métricas escala dinámicamente con el estrés sistémico:

\[
\mu_{final} = 1 + (\mu_{base} - 1) \cdot (1 + \lambda \cdot S)
\]

### 6. Operadores de Mutación Matricial

El `NetworkActor` aplica los impactos directamente sobre las matrices dispersas:
- **Corte Topológico**: \(A_{uv} \leftarrow 0, K_{uv} \leftarrow 0\)
- **Degradación de Capacidad**: \(K_{uv} \leftarrow K_{uv} \cdot \beta_{cap}\)
- **Inflación de Métricas**: \(C_{uv} \leftarrow C_{uv} \cdot \mu_{final}(\gamma, S)\)

### 7. Algoritmo de Generación Monte Carlo

1. **Clonación**: Copia profunda de \(\mathcal{N}_0\).
2. **Propagación**: Iteración en orden topológico.
3. **Activación Estocástica**: Se evalúa \(r \sim U[0,1]\). Si \(r < P_{eff}\), el evento se activa.
4. **Acumulación de Estrés**: \(S \leftarrow S + weight(E)\).
5. **Mutación**: Deformación de las matrices.

### 8. Criterio de Convergencia Estadística

El motor monitorea la estabilidad en ventanas deslizantes para evitar sobrecómputo:

\[
|\mu_{window} - \mu_{prev}| < \epsilon \quad \land \quad |\sigma^2_{window} - \sigma^2_{prev}| < \epsilon
\]

### 9. Salida del Bloque

El resultado es un objeto serializado que contiene:
- El **Conjunto de Escenarios**: \(\mathcal{S} = \{Scenario_1, ..., Scenario_N\}\).
- Metadatos y Estadísticas Agregadas.

Este conjunto \(\mathcal{S}\) constituye la entrada para el bloque III.

---

## Bloque III — Inferencia de Riesgo Estructural (Ortogonal)

### 1. Propósito del Bloque

El **Bayesian Auditor** actúa como el tribunal forense del sistema. Su función es procesar la evidencia empírica generada por el bloque II para transformar "datos de simulación" en "conocimiento de confiabilidad".

A diferencia de modelos obsoletos que colapsan probabilidad y costo monetario en un escalar único (caja negra), este bloque respeta la dimensionalidad física del riesgo. Calcula la **probabilidad de fallo** y el **impacto esperado** de manera estrictamente separada y ortogonal, preparando el terreno tensorial para la optimización multiobjetivo.

### 2. Auditoría Forense de Escenarios

El primer paso es determinista. El módulo **Auditor** somete cada escenario simulado \(S_k\) a un juicio binario basado en `FailureCriteria`.

Un escenario se declara **EXITOSO** (\(Y_k = 1\)) si y solo si cumple simultáneamente:
- **Integridad de Capacidad**: \(K_{retained} \ge 85\%\)
- **Estabilidad de Tiempos**: \(T_{travel} \le 1.4 \times T_{base}\)
- **Eficiencia de Costos**: \(C_{total} \le 1.5 \times C_{base}\)

Si alguna métrica viola el umbral, el escenario se marca como **FALLIDO** (\(Y_k = 0\)) y se registran los componentes causales.

### 3. Modelo de Inferencia Beta-Binomial

Para inferir la confiabilidad latente \(\theta_u\) de cada componente \(u \in V \cup E\), utilizamos el modelo conjugado **Beta-Binomial**.

#### a. Priors conjugados
Asumimos una creencia inicial sobre la confiabilidad \(\theta_u\) (probabilidad de éxito):

\[
\theta_u \sim Beta(\alpha_0, \beta_0)
\]

#### b. Actualización Bayesiana
Al observar \(N\) escenarios, acumulamos éxitos (\(s_u\)) y fallos (\(f_u\)). La distribución posterior es analítica exacta:

\[
\theta_u | Data \sim Beta(\alpha_0 + s_u, \beta_0 + f_u)
\]

### 4. Desacople Dimensional del Riesgo (Probabilidad e Impacto)

En lugar de construir métricas de "fragilidad compuesta" multiplicando variables incompatibles, el `BayesianJudge` extrae dos métricas fundamentales y ortogonales para cada componente \(u\):

1. **Probabilidad de fallo posterior** (\(P(F_u)\)):
   La probabilidad pura de que el componente colapse.
   \[
   P(F_u) = 1 - E[\theta_u] = 1 - \frac{\alpha_{post}}{\alpha_{post} + \beta_{post}}
   \]

2. **Impacto promedio condicional** (\(\bar{I}_u\)):
   El daño medio al sistema (fricción o costo adicional) observado exclusivamente en los escenarios donde el componente \(u\) falló.

### 5. Salida: Reliability Report Ortogonal

El Bloque III rechaza la creación de una matriz de riesgo unificada. En su lugar, el `InferenceEngine` emite un **ReliabilityReport** que entrega colecciones de datos puros por componente:

Para cada arco y nodo, el reporte entrega el par desacoplado:

\[
\{P(F_u), \bar{I}_u\}
\]

Además de proveer los **Intervalos de Confianza** (varianza \(\sigma^2\)) para distinguir entre riesgo conocido y genuina incertidumbre sistémica. Estos datos crudos son el sustrato que el Bloque IV utilizará para proyectar los tensores espaciales.

---

## Bloque IV — Optimización Estratégica Multiobjetivo (V1.3)

### 1. Cambio de paradigma: Del Dijkstra Escalar a la Búsqueda Multiobjetivo

La optimización tradicional (Dijkstra) colapsa todas las variables en un solo escalar alterando los pesos mediante un parámetro artificial (\(\kappa\)). Prime Logistics V1.3 abandona esta heurística.

El **MultiObjective Engine** trata el ruteo como un genuino problema geométrico de múltiples dimensiones en conflicto: **Supervivencia** (Seguridad) vs. **Costo Esperado** (Eficiencia). El objetivo del bloque no es hallar una ruta, sino descubrir la **Frontera de Pareto exacta** en una sola pasada de cómputo, revelando todos los trade-offs matemáticamente posibles.

### 2. Preparación Espacial: Tensores Ortogonales

Antes de la búsqueda, el `weight_builder` proyecta el grafo físico hacia un espacio tensorial de dos dimensiones utilizando los datos ortogonales entregados por el Bloque III.

Para cada arco \(e_{ij}\), se instancian dos pesos irreductibles:

#### Eje 1: Peso de Supervivencia (\(W^{(1)}\))

Dado que la probabilidad conjunta de un camino es multiplicativa (\(P_{ruta} = \prod (1 - P(F_i))\)), se aplica una transformación isomorfa al espacio logarítmico para volverla aditiva y compatible con algoritmos de grafos:

\[
W_{ij}^{(1)} = -\ln(1 - P(F_{ij}))
\]

(Minimizar \(W^{(1)}\) es matemáticamente idéntico a maximizar la supervivencia de la ruta).

#### Eje 2: Costo Esperado Operativo (\(W^{(2)}\))

Se incorpora el costo base monetario/temporal más el daño friccional esperado en caso de fallo:

\[
W_{ij}^{(2)} = C_{ij} + P(F_{ij}) \times \bar{I}_{ij}
\]

(Minimizar \(W^{(2)}\) asegura eficiencia de capital bajo incertidumbre).

### 3. El Motor de Label Setting y Dominancia \(\epsilon\)

El núcleo del sistema es un algoritmo de **Multi-Objective Label Setting**. A diferencia de Dijkstra, un nodo no almacena la "mejor" distancia, sino un conjunto de **Etiquetas (Labels)** no dominadas. Una etiqueta \(L_a\) domina a \(L_b\) si es mejor o igual en ambos ejes (\(W^{(1)}\) y \(W^{(2)}\)) y estrictamente mejor en al menos uno.

Para evitar la explosión combinatoria (la maldición de la dimensionalidad en grafos densos), el motor implementa **\(\epsilon\)-dominancia espacial**.

El espacio de soluciones de cada nodo se discretiza en una cuadrícula mediante dos tolerancias paramétricas (\(\epsilon_1\) para supervivencia y \(\epsilon_2\) para costo monetario):

\[
Box = \left( \lfloor \frac{W^{(1)}}{\epsilon_1} \rfloor , \lfloor \frac{W^{(2)}}{\epsilon_2} \rfloor \right)
\]

El algoritmo aniquila en tiempo \(\mathcal{O}(1)\) cualquier etiqueta que caiga en una "caja" previamente conquistada por una ruta más eficiente, logrando un **Frente de Pareto \(\epsilon\)-aproximado** que protege la memoria sin perder resoluciones estratégicas.

### 4. Perfilado Estructural Analítico (Route Profiler)

Tras extraer las rutas supervivientes en el nodo de destino, el sistema ejecuta una "biopsia estructural" para caracterizar la calidad de la topología elegida:

#### a. Entropía Relativa (Incertidumbre de Shannon)

Mide la distribución del riesgo a lo largo de la ruta.

\[
H_{rel}(R) = \frac{-\sum p_i \log_2 p_i}{\log_2 |R|}
\]

- \(H\) baja (< 0.3): Riesgo concentrado (frágil).
- \(H\) alta (> 0.7): Riesgo distribuido uniformemente (robusto).

#### b. Índice de Rigidez

Una combinación lineal de vulnerabilidades que penaliza la exposición nodal, la alta volatilidad de costos y la falta de alternativas viables (Redundancia Estructural).

### 5. Extracción Euclidiana de Arquetipos Estratégicos

El módulo `pareto.py` no le entrega matemáticas crudas al usuario. Analiza geométricamente la Frontera de Pareto resultante y clasifica las soluciones en **arquetipos de negocio**, listos para la toma de decisión:

- **El Tanque**: Mínimo absoluto en el eje de Supervivencia (\(W^{(1)}\)). La opción "Zero-Trust", prioriza la seguridad atómica por sobre el capital.
- **El Apostador**: Mínimo absoluto en el eje de Costo Esperado (\(W^{(2)}\)). Máxima eficiencia de flujo de caja, pero asume riesgos estructurales severos.
- **El Equilibrista**: Mediante la normalización min-max de los extremos del frente, halla la solución con la mínima distancia Euclidiana al punto utópico \((0,0)\). El balance operativo perfecto.
- **El Unicornio**: Detectado mediante una regla de negocio condicional. Si la diferencia porcentual de costo (sobrecosto) entre El Tanque y El Apostador es estadísticamente nula, la opción se clasifica como una anomalía de mercado de dominancia total (Bajo costo y Máxima seguridad).

### 6. Salida del Sistema: Prime Strategic Report

El orquestador finaliza el ciclo entregando un reporte narrativo de inteligencia táctica. Traduce la matemática de \(\epsilon\)-dominancia y las entropías en una recomendación de acción directa ("Ejecutar", "Monitorear", "Descartar") junto con un porcentaje de **Confianza del Sistema** derivado de los pesos estructurales, culminando la evolución de Prime Logistics hacia una arquitectura **Deep-Tech empresarial**.

---

## Limitaciones Conocidas y Suposiciones

Para mantener la viabilidad computacional en este MVP, el modelo acepta los siguientes trade-offs teóricos:

1. **Independencia ingenua en la inferencia**: La actualización Beta-Binomial asume intercambiabilidad de las ejecuciones de simulación. Si bien se generan fallas correlacionadas (cascadas), el paso de inferencia trata la evidencia como pseudo-independiente para calcular la probabilidad local. Esto puede llevar a una confianza excesiva en el posterior para redes altamente acopladas.
2. **Enfoque Estructural vs. Operacional**: El modelo minimiza el riesgo estructural (disponibilidad física de conexiones) en lugar de la latencia operacional (retrasos en colas). Actualmente no implementa dinámicas estocásticas de colas (M/G/k) en los nodos.
3. **Flujo estático**: El optimizador multiobjetivo asume enrutamiento estático preventivo antes del inicio del viaje, ignorando las capacidades de re-enrutamiento dinámico (Markov Decision Processes) de los agentes durante el evento de falla en tiempo real.
