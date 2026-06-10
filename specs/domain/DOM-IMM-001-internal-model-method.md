---
id: DOM-IMM-001
title: "Internal Model Method (IMM): Conceptos Fundamentales"
type: knowledge
layer: domain
status: active
confidence: high
version: 1.0.0
owner: risk-team
domain: internal-model-method
subdomain: counterparty-credit-risk
tags: [IMM, CCR, methodology, BBVA]
uses-data-from: []
constrained-by: []
supersedes: []
---

## Intent

Establecer una definición clara de qué es el Internal Model Method (IMM) en el contexto de BBVA, sus principios fundamentales, y cómo se aplica en el cálculo de riesgo de crédito de contrapartida (CCR).

## Definition

### ¿Qué es IMM?

El **Internal Model Method** es un enfoque regulatorio (según Basilea III) que permite a las instituciones financieras utilizar sus propios modelos internos para calcular la **exposición al riesgo de crédito de contrapartida (CCR)**.

En BBVA, IMM se utiliza para:
- Calcular la **exposición esperada futura** en derivados y contratos de financiamiento
- Determinar los **requisitos de capital** basados en modelos propios validados
- Aplicar **ajustes de valoración crediticia (CVA)** a posiciones de derivados

### Principios Fundamentales

**Exposición Esperada Futura (EE - Expected Exposure)**
- Medida estadística de la exposición máxima probable en un horizonte futuro
- Se calcula con confidence level del **95% o 99%** según regulación
- Varía en el tiempo según cambios en el valor de mercado del contrato

**Componentes del Cálculo:**
1. **Simulación de Trayectorias**: Múltiples caminos de evolución futura del mercado
2. **Valoración Forward**: Valor del contrato en cada fecha de simulación
3. **Agregación Neteable**: Aplicación de CSA (Credit Support Annex)
4. **Medida de Exposición**: EE, PFE (Potential Future Exposure), EEPE (Effective Expected Exposure)

### Ventajas respecto a métodos alternativos

| Aspecto | IMM | Enfoque Estándar |
|--------|-----|------------------|
| **Sensibilidad al Riesgo** | Alta - adapta a cada cartera | Baja - fórmulas genéricas |
| **Requisitos de Capital** | Optimizados según perfil real | Conservadores, generalizados |
| **Flexibilidad** | Captura netting agreements | Limitada a agregaciones simples |

## Acceptance Criteria

**AC-IMM-01**: El modelo IMM debe estar validado por la función de Riesgos de BBVA antes de su uso en cálculos de capital.

**AC-IMM-02**: Toda exposición calculada por IMM debe ser rastreable al contrato o transacción subyacente.

**AC-IMM-03**: El sistema debe permitir recalcular exposiciones para cualquier fecha histórica con los datos disponibles en esa fecha.

**AC-IMM-04**: Los parámetros de entrada (volatilidades, correlaciones) deben actualizarse mínimo mensualmente.

## Evidence

- Documento de Metodología: "Internal Model Method for Counterparty Credit Risk BBVA - Methodology"
- Validación Regulatoria: Basilea III - Artículos sobre CCR
- Implementación Vigente: Sistema de Risk Management BBVA (desde 2015)

## Related Specs

- `DOM-BT-001`: Backtesting - propósito y alcance
- `ARCH-IMM-001`: Integración técnica del módulo IMM
