# PRIME LOGISTICS | Stochastic Risk & Topology Engine

## Cuantificación de Fragilidad en Redes de Suministro mediante Modelado de Cascada Jerárquica

---

## 1. Resumen Ejecutivo (The "Why")

En logística industrial, la mayoría de los sistemas de planificación fallan porque asumen condiciones ideales y estáticas. **PRIME LOGISTICS** no es simplemente un sistema de ruteo; es un **motor de estrés operacional** que cuantifica el riesgo oculto dentro de su infraestructura de suministro.

Mediante una arquitectura de simulación **Monte Carlo con Convergencia Dual**, el motor genera miles de "escenarios de degradación" para identificar no solo los puntos de falla de la red, sino también cómo se ven afectados los costos y tiempos de entrega bajo presión sistémica.

**Propósito principal:** Permitir que las organizaciones midan y comprendan la resiliencia real de sus redes logísticas antes de que ocurra una crisis.

---

## 2. Capacidades Actuales (The "What")

### A. Construcción de Topología Multidimensional (Bloque 1)

Transforma datos operativos en bruto (coordenadas, capacidades, perfiles de costo) en un **modelo de cinco matrices equilibradas**:

| Matriz/Variable | Descripción |
|----------------|-------------|
| **Matriz de Adyacencia (A)** | Representa la conectividad física de la red (qué nodos están vinculados). |
| **Matriz de Costo (C)** | Costos de transporte por arco, basados en distancias reales (fórmula de Haversine) y tarifas. |
| **Matriz de Tiempo (T)** | Tiempos de tránsito por arco, utilizando perfiles de velocidad estocásticos. |
| **Matriz de Capacidad (K)** | Límites físicos de flujo (volumen/peso) por cada conexión. |
| **Vector de Demanda (D)** | Balance de carga entre nodos de origen y destino. |

### B. Motor Estocástico de Cascada (Bloque 2)

Implementa un **modelo de propagación de riesgo jerárquico** que simula la realidad de mercados volátiles:

1.  **Nivel Sistémico (Macro):** Eventos disruptivos mayores ("Cisnes Negros") que activan un **Índice de Estrés (S)**.
2.  **Nivel Táctico (Red):** Fallos en nodos o conexiones específicas, cuya probabilidad de ocurrencia depende del estado sistémico.
3.  **Nivel Operativo (Micro):** Ruido y variabilidad diaria (ej: tráfico, clima local) cuya intensidad se escala según el nivel de degradación ambiental.

---

## 3. Fundamentación Matemática (The "Logic")

### El Modelo de Intensidad de Riesgo

El núcleo del sistema es un **Índice de Estrés Agregado (S)**, que modifica dinámicamente los parámetros de la red en cada iteración de la simulación:

```
S = Σ (wₖ · I{Sₖ activo})
```

**Donde:**
*   `wₖ` = Severidad estructural del evento *k*.
*   `I{Sₖ activo}` = Función indicadora (1 si el evento sistémico *k* está activo, 0 si no).

Basándose en `S`, las métricas clave de la red (como el tiempo medio de tránsito) se **recalculan en tiempo real**:

```
μ_efectivo = μ_base · (1 + α · S)
σ_efectivo = σ_base · (1 + β · S)
```

**Donde:**
*   `μ_efectivo` = Media ajustada (ej: tiempo promedio de entrega).
*   `σ_efectivo` = Desviación estándar ajustada (incertidumbre operativa).
*   `α`, `β` = Factores de sensibilidad al estrés.

### Condición de Diseño Fundamental

El sistema garantiza que **`β > α`**. Esto significa que, en una crisis, la **incertidumbre (`σ`)** crece más rápido que el **retraso promedio (`μ`)**.

**Interpretación:** Modela matemáticamente la **pérdida de predictibilidad y control operativo** característica de situaciones de crisis logística, donde los tiempos no solo empeoran, sino que se vuelven impredecibles.

# PRIME LOGISTICS | Stochastic Risk & Topology Engine

## 4. Arquitectura del Sistema

El sistema está estructurado en **tres bloques modulares** que operan en secuencia:

```
┌─────────────────┐    ┌───────────────────┐    ┌─────────────────────┐
│   BLOQUE 1      │    │    BLOQUE 2       │    │    BLOQUE 3        │
│   Topología     │───▶│   Motor de Riesgo │───▶│   Optimización     │
│   Multidimensional  │    │   Estocástico     │    │   (Futuro)         │
└─────────────────┘    └───────────────────┘    └─────────────────────┘
       │                         │                         │
       ▼                         ▼                         ▼
   Generación de            Simulación de            Planificación de
   Modelo Estático          Escenarios de            Mitigación de
   (A, C, T, K, D)          Cascada                  Riesgo
```

**Flujo de datos:** El **Bloque 1** transforma datos logísticos crudos en un modelo de red estructurado. Este modelo alimenta al **Bloque 2**, donde se somete a miles de perturbaciones simuladas. Los resultados de estas simulaciones (distribuciones de costos, tiempos y puntos de falla críticos) están diseñados para informar las estrategias del **Bloque 3**.

---

## 5. Rigor de Implementación (Tech Specs)

### Manejo de Matrices Dispersas
- **Tecnología:** Implementado con `scipy.sparse`.
- **Formatos:** Uso estratégico de formatos **CSR** (operaciones eficientes) y **LIL** (construcción flexible).
- **Objetivo:** Permite modelar redes a gran escala (> 10,000 nodos) con eficiencia de memoria O(n), evitando el cuello de botella de matrices densas O(n²).

### Convergencia Estocástica Dual
El motor no usa un número de iteraciones fijo. En su lugar, implementa un **criterio de parada adaptativo** basado en la estabilidad estadística de la simulación:
```
(Δσ² / σ²) < ε
```
**Donde:**
- `Δσ²` = Cambio en la varianza de la métrica observada entre lotes de simulaciones.
- `σ²` = Varianza actual de la métrica.
- `ε` = Umbral de tolerancia predefinido (ej: 0.001).

**Ventaja:** Garantiza que los resultados sean estadísticamente representativos sin ciclos de cómputo innecesarios.

### Blindaje de Integridad
- **Mecanismo:** El sistema incluye capas de resiliencia interna para manejar estados de red degradados.
- **Función:** Gestiona automáticamente el *fallback* entre métricas cuando encuentra valores extremos (infinitos) o desconexiones totales del grafo. Prioriza el mantenimiento de la conectividad (`A`) sobre la capacidad (`K`) en escenarios de falla crítica.

---

## 6. Estado del Proyecto

| Bloque | Nombre | Estado | Notas |
|--------|---------|---------|-------|
| **1** | Topología Multidimensional |  **100% Finalizado** | Generación estable del modelo de 5 matrices (A, C, T, K, D). |
| **2** | Motor de Riesgo Estocástico |  **100% Finalizado** | Simulación de cascada con convergencia dual operativa. |
| **3** | Optimización & Mitigación |  **En Diseño Conceptual** | Definición de la "Función Objetivo de Mínimo Arrepentimiento". |

---

## 7. Aviso de Propiedad Intelectual

 **CÓDIGO PRIVADO | DOCUMENTACIÓN PÚBLICA**

- **IP Privada:** La arquitectura del motor de riesgo, los algoritmos de mutación matricial y el núcleo de simulación son **Propiedad Intelectual (IP) privada**.
- **Propósito del Repositorio:** Este espacio expone la lógica técnica, los fundamentos matemáticos y las capacidades del sistema para fines de **auditoría, colaboración académica y demostraciones de concepto**. No contiene el código fuente ejecutable completo.

---

## 8. Contacto

**Ariel Duarte** - Engineering Student   
- **LinkedIn:** www.linkedin.com/in/arielduarte-j 
- **Email:** Arielduartejesus@gmail.com  

