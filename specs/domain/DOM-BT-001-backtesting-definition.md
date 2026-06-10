---
id: DOM-BT-001
title: "Backtesting: Definición, Propósito y Alcance"
type: knowledge
layer: domain
status: active
confidence: high
version: 1.0.0
owner: risk-team
domain: backtesting
subdomain: model-validation
tags: [backtesting, validation, CCR, IMM]
uses-data-from: [DOM-IMM-001]
constrained-by: []
supersedes: []
---

## Intent

Definir qué es el backtesting en el contexto de los modelos IMM de BBVA, su propósito regulatorio y de negocio, y el alcance exacto de la validación que se realiza.

## Definition

### ¿Qué es Backtesting?

**Backtesting** es la comparación de **exposiciones calculadas (predicted)** por el modelo IMM contra **exposiciones reales observadas (realized)** en el pasado, con el objetivo de validar que el modelo produce estimaciones razonables.

### Propósito del Backtesting

**Regulatorio:**
- Cumplir con requisitos de validación de modelos internos (Basilea III)
- Documentar la robustez del modelo IMM para autoridades regulatorias
- Justificar el uso de capital optimizado basado en modelos propios

**De Negocio:**
- Detectar desviaciones sistemáticas en el modelo
- Identificar periodos o contrapartidas donde el modelo es débil
- Validar parámetros de entrada (volatilidades, correlaciones)
- Mantener confianza en cálculos de capital y provisiones

### Tipos de Backtesting en BBVA

**1. Backtesting de Exposición (Exposure Backtesting)**
- Compara EE (Expected Exposure) predicha vs. realizada
- Métrica: Número de excepciones (ocasiones donde la exposición real > EE predicho)
- Threshold: <5% de excepciones anuales es aceptable

**2. Backtesting de CVA (CVA Backtesting)**
- Valida el ajuste de valoración crediticia
- Compara cambios predichos vs. realizados en CVA
- Métrica: Correlación entre predicción y realidad

**3. Backtesting de PFE (Potential Future Exposure)**
- Evalúa si el percentil 95% o 99% es apropiado
- Verifica que no hay excesos anómalos

## Acceptance Criteria

**AC-BT-01**: El backtesting debe cubrir un período mínimo de 1 año para ser considerado válido.

**AC-BT-02**: La tasa de excepciones (exposición real > predicha) no debe exceder el 5% anual para exposiciones en el percentil 95%.

**AC-BT-03**: Cada excepción debe ser documentada y analizada para identificar causas (market stress, modelo weakness, data issues).

**AC-BT-04**: El análisis de backtesting debe incluir segregación por:
- Tipo de contrapartida (bank, corp, fund)
- Tipo de producto (derivatives, repo, other)
- Geografía

**AC-BT-05**: Los resultados de backtesting deben ser reportados mensualmente a la función de Riesgos.

## Evidence

- Documento: "C102 - Backtesting v1.0"
- Metodología: "Internal Model Method for Counterparty Credit Risk BBVA - Methodology"
- Regulación: Basilea III - Comprehensive Risk Measure framework

## Related Specs

- `DOM-IMM-001`: Internal Model Method - conceptos fundamentales
- `DOM-BT-002`: Backtesting - metodología de cálculo
- `ARCH-BT-001`: Generador backtesting - arquitectura
