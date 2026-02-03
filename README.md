# PRIME LOGISTICS | Stochastic Optimization Engine
![Python](https://img.shields.io/badge/Python-3.10%2B-blue) ![Status](https://img.shields.io/badge/Status-Prototype-orange) ![License](https://img.shields.io/badge/License-Proprietary-red)

**Resiliencia de Redes de Distribución bajo Incertidumbre Estructural mediante Simulación de Cascada.**

---

## 1. Resumen Ejecutivo (The "Why")
En la logística de alta complejidad, los promedios son una trampa. Los modelos tradicionales optimizan para un "escenario ideal", ignorando la volatilidad de entornos reales. Cuando ocurre un evento sistémico (conflictos gremiales, crisis de insumos o colapsos climáticos), las cadenas de suministro fallan no por falta de capacidad, sino por falta de robustez.

**PRIME LOGISTICS** desacopla la topología estática de la dinámica de riesgo. Mediante un motor de Monte Carlo con convergencia dual, generamos miles de "futuros posibles" para encontrar la **ruta de mínimo arrepentimiento**, transformando la incertidumbre en una variable de diseño controlable.

---

## 2. Fundamentación Matemática (The "Logic")

### Función Objetivo: Optimización Robusta
El motor no busca simplemente minimizar el costo, sino el valor esperado ponderado por la aversión al riesgo del decisor:

$$\min Z = E \left[ \sum_{i,j \in \mathcal{A}} C_{\omega, ij} \cdot x_{ij} \right] + \lambda \cdot \text{VaR}_{\alpha}(\text{Loss})$$

Donde:
* $C_{\omega, ij}$: Costo del arco $(i, j)$ en el escenario estocástico $\omega$.
* $\lambda$: Coeficiente de aversión al riesgo.
* $\text{VaR}_{\alpha}$: Valor en Riesgo al nivel de confianza $\alpha$.

### Modelado de Incertidumbre y Cascada
El caos se propaga mediante un **Stress Index ($S$)** que escala las magnitudes de los eventos de nivel inferior:

$$S = \sum_{k \in \text{Sistémicos}} w_k \cdot \mathbb{1}_{\{S_k \text{ activo}\}}$$

Bajo condiciones de estrés ($S > 0$), la varianza de los tiempos y costos se degrada más rápido que la media, reflejando la explosión de incertidumbre operativa:

$$\mu_{eff} = \mu_{base} \cdot (1 + \alpha \cdot S)$$
$$\sigma_{eff} = \sigma_{base} \cdot (1 + \beta \cdot S)$$

> **Restricción de Diseño:** $\beta > \alpha$ (La incertidumbre crece más rápido que el costo medio).

---

## 3. Arquitectura del Sistema
El sistema se basa en una arquitectura de **Actor-Mutador**, garantizando que la red original nunca se corrompa y que cada escenario sea trazable.

```mermaid
graph TD
    A[Bloque 1: Topology Builder] -->|Pentada Base A,C,T,K,D| B[Bloque 2: Stochastic Engine]
    C[Manifest.yaml: ADN de Riesgo] --> B
    B --> D{Monte Carlo Loop}
    D -->|Estabilidad de Media y P95| E[PossibleTomorrow Scenarios]
    E -->|scenarios.pkl| F[Bloque 3: Robust Optimizer]
    F --> G[Ruta de Mínimo Arrepentimiento]

4. Resultados y Validación
El sistema ha sido validado mediante pruebas de estrés contra "Cisnes Negros" logísticos.

(Figura 1: Comparación de densidad de probabilidad de costos. La curva azul (Estándar) subestima el riesgo de cola que la curva roja (Prime Logistics) captura).

Identificación de Cuellos de Botella Estocásticos: El modelo detectó que, ante una crisis de combustible, la capacidad de la red se degrada en un 20% debido a desviaciones operativas, incrementando la varianza de los tiempos de entrega en un 35%.

Convergencia Dual: El motor alcanzó estabilidad estadística en 200 iteraciones, garantizando representatividad sin desperdicio de cómputo.

5. Tech Stack
Core: Python 3.10+

Computación Numérica: NumPy & SciPy (Estructuras de matrices dispersas para alta dimensionalidad).

Validación de Datos: Pydantic / Dataclasses para cumplimiento de contratos de red.

Simulación: Monte Carlo Engine con lógica de cascada condicional.

6. Aviso de Propiedad Intelectual
⚠️ CÓDIGO PRIVADO | DOCUMENTACIÓN PÚBLICA

El código fuente que implementa el motor de mutación matricial y las heurísticas de optimización es Propiedad Intelectual (IP) privada y constituye un prototipo comercial. La presente documentación se ofrece con fines de exposición de la lógica arquitectónica y robustez matemática.

7. Contacto
Ariel Duarte - Engineering & Deep-Tech Development

LinkedIn

Email: Arielduartejesus@gmail.com
