---
id: DOM-BT-002
title: "Backtesting: Metodología de Cálculo y Métricas"
type: knowledge
layer: domain
status: active
confidence: high
version: 1.0.0
owner: risk-team
domain: backtesting
subdomain: calculation-methodology
tags: [backtesting, methodology, metrics, calculation]
uses-data-from: [DOM-BT-001, DOM-IMM-001]
constrained-by: []
supersedes: []
---

## Intent

Especificar cómo se calcula el backtesting de exposición en BBVA: qué datos se usan, qué fórmulas se aplican, y cómo se interpreta el resultado.

## Definition

### Fórmula de Cálculo de Exposición

Para cada contrapartida en una fecha de observación:

```
EE_t(i) = E[ max(0, V_t(i)) ]
```

Donde:
- **E[·]** = esperanza matemática (media) sobre todas las trayectorias simuladas
- **V_t(i)** = valor del portafolio de la contrapartida i en la fecha t
- **max(0, ·)** = aplicación del principio de **optionality** (solo hay pérdida si V > 0 para BBVA)

### Proceso de Backtesting

**Paso 1: Selección de Fechas de Observación**
- Frecuencia: Mensual, mínimo
- Horizonte: Últimos 12-24 meses de histórico
- Cada fecha debe tener al menos 50 contrapartidas activas

**Paso 2: Recolección de Exposición Predicha**
- Usar EE (Expected Exposure) calculada en fecha t-k (ej. t-1 mes)
- Usar percentil 95% o 99% según regulación
- Segregar por tipos de contrapartida

**Paso 3: Cálculo de Exposición Realizada**

```
EE_realizado = Valor del portafolio en fecha t con datos de mercado de fecha t
```

Datos necesarios:
- Precios de spots (FX, rates, commodities)
- Volatilidades implícitas de opciones
- Correlaciones de mercado
- Ajustes por colateral (CSA)

**Paso 4: Identificación de Excepciones**

```
Excepción = 1 si EE_realizado > EE_predicho
Excepción = 0 en caso contrario
```

**Paso 5: Cálculo de Tasa de Excepciones**

```
Tasa_Excepciones = (Num_Excepciones) / (Total_Observaciones) × 100%
```

### Análisis de Excepciones

Para cada excepción, registrar:

| Campo | Descripción | Ejemplo |
|-------|-------------|----------|
| **Fecha** | Fecha de observación | 2024-01-31 |
| **Contrapartida** | ID de contrapartida | CORP-001 |
| **Tipo** | Tipo de contrapartida | Corporate |
| **EE_Predicho** | Exposición predicha (USD M) | 50.2 |
| **EE_Realizado** | Exposición realizada (USD M) | 62.5 |
| **Exceso** | Diferencia | 12.3 |
| **Causa Probable** | Categoría de raíz | Market Stress, Vol Jump, FX Move |
| **Acción** | Acción tomada | Monitor, Investigate, Model Change |

### Métrica de Validación

**Green Zone** (Aceptable):
- Tasa de excepciones: 0% - 5%
- Distribución uniforme en tiempo
- Sin concentración en tipo de contrapartida

**Yellow Zone** (Atención):
- Tasa de excepciones: 5% - 8%
- Requerir investigación y plan de corrección

**Red Zone** (Inaceptable):
- Tasa de excepciones: > 8%
- Modelo debe ser revisado/recalibrado antes de usar en cálculos de capital

## Acceptance Criteria

**AC-BT-01**: La fórmula de EE debe aplicarse de forma consistente a todas las contrapartidas.

**AC-BT-02**: Los datos de mercado para exposición realizada deben ser de EOD (End of Day) estándar.

**AC-BT-03**: Toda excepción debe documentarse en el registro de backtesting con causa probable y acción.

**AC-BT-04**: La tasa de excepciones debe calcularse y reportarse segregada por:
- Tipo de contrapartida (bank, corp, fund, other)
- Región geográfica
- Moneda de exposición

**AC-BT-05**: Si la tasa de excepciones entra en Yellow o Red Zone, debe iniciarse un plan correctivo documentado.

## Evidence

- P036: Generador Backtesting - especifica cálculos
- C102: Backtesting v1.0 - metodología detallada
- Basilea III: Comprehensive Risk Measure

## Related Specs

- `DOM-BT-001`: Backtesting - definición y alcance
- `DOM-IMM-001`: Internal Model Method - conceptos
- `ARCH-BT-001`: Generador backtesting - arquitectura
