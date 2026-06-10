---
id: PROD-BT-001
title: "Backtesting: Product Features and User Workflows"
type: knowledge
layer: product
status: active
confidence: high
version: 1.0.0
owner: product-team
domain: backtesting
subdomain: product-features
tags: [product, backtesting, validation, user-workflows]
uses-data-from: [DOM-BT-001, DOM-BT-002]
constrained-by: []
supersedes: []
---

## Intent

Definir las características del producto de Backtesting desde la perspectiva del usuario (Risk Officer, Model Governance, Regulators): cómo se genera, cómo se interpreta, y qué acciones dispara.

## Definition

### Value Proposition

**Para Risk Officers:**
- ✅ Validar automáticamente que el modelo IMM es preciso
- ✅ Identificar periodos o contrapartidas donde el modelo falla
- ✅ Documentar validación para reguladores
- ✅ Trigger correcciones de modelo cuando es necesario

**Para Model Governance:**
- ✅ Monitorear salud del modelo de forma continua
- ✅ Clasificar excepciones por causa (market stress vs. model weakness)
- ✅ Quantificar impacto de cambios de modelo
- ✅ Producir reportes de cumplimiento regulatorio

**Para Regulators:**
- ✅ Confirmar que banco valida sus modelos internos
- ✅ Revisar tasa de excepciones y causas
- ✅ Evaluar rigor de validación pre-capital calculation

### Capacidades Principales

#### 1. Automatic Weekly Backtesting

**Qué hace:**
- Extrae predicciones históricas (EE de hace 30 días)
- Reval pautas actuales con market data de hoy
- Compara: realizado > predicho? = excepción
- Calcula tasa de excepciones
- Genera alert si entra en Yellow/Red zone

**Frequency:**
- Weekly (automation)
- Daily (on-demand)

**Data scope:**
- 24 meses de histórico
- Mínimo 50 contrapartidas por período
- Segregación por: tipo, región, moneda

#### 2. Exception Analysis & Reporting

**Qué hace:**
- Para cada excepción, registra:
  - Fecha de observación
  - Contrapartida
  - EE predicho vs. realizado
  - Porcentaje de exceso
  - Causa probable (market stress, vol jump, FX move, model weakness)
  - Acción recomendada (monitor, investigate, adjust)

**Output report:**
- Summary table: Tasa de excepciones por segmento
- Gráficos: Excepciones en tiempo, distribución por tipo
- Top 10 contrapartidas con excepciones
- Análisis de causas
- Traffic light status (Green/Yellow/Red)
- Recomendaciones de acción

#### 3. Model Health Dashboard

**Qué hace:**
- Visualiza métricas de validación en tiempo real
- Trending de tasa de excepciones
- KPI monitoring (excepciones, concentración, correlación)
- Drill-down a nivel de excepción individual

**Metrics displayed:**
- Exception rate (current, 12M average, trend)
- Green/Yellow/Red zone status
- Top failing counterparties
- Top failing products
- Exception causes breakdown (pie chart)
- Correlation scatter: predicted vs. realized
- Concentration of exceptions (top 5)

#### 4. Root Cause Investigation Tools

**Qué hace:**
- Para cada excepción, permite investigar:
  - Market data que movió el precio
  - Volatilidad/correlaciones del día
  - Sensibilidades de los derivados
  - Cambios en CSA/collateral
  - Settlement o default events

**Features:**
- Side-by-side comparison: predicción vs. realidad
- Market data explorer (rates, FX, vols)
- Sensitivity analysis (Greeks)
- Scenario simulation: "what if vol was +10%"

#### 5. Regulatory Submission Support

**Qué hace:**
- Genera reportes en formato regulatorio
- Evidencia de validación para Banco de España/ECB
- Traceability de cálculos (audit trail)
- Compliance checking (tasa excepciones vs. thresholds)

**Report types:**
- Regulatory validation report (monthly/quarterly)
- Supervisor submission package
- Independent review evidence

### User Journeys

#### Journey 1: Weekly Model Validation (Risk Officer)

```
Every Monday, 10:00 AM:
  - System auto-runs backtesting over weekend
  - Report ready for review
  
10:15 AM: Risk Officer opens Backtesting Dashboard
  - Sees: Exception rate = 3.2% (Green zone ✅)
  - Sees: No new alerts
  - Sees: Top exception: Goldman Sachs (2 exceptions in month)
  
10:30 AM: Drills into Goldman exceptions
  - Date 1: Exceso 8%, Causa: "FX volatility spike"
  - Date 2: Exceso 12%, Causa: "Market stress event"
  - Assessment: Normal market behavior, not model weakness
  
10:45 AM: Approves weekly validation
  - Signs off: "Model performance satisfactory"
  - Notifies team: No action required
```

#### Journey 2: Yellow Zone Trigger & Investigation (Model Governance)

```
Weekly backtesting completes:
  - Exception rate = 6.8% (Yellow zone ⚠️)
  - System auto-alerts Model Governance team
  
09:00 AM (Day+1): Model Governance opens investigation
  - Filters exceptions by cause
  - Sees: 60% = "Vol Jump", 25% = "Model weakness", 15% = "Market stress"
  - Hypothesis: Volatility module is miscalibrated
  
10:00 AM: Deep dive into volatility module
  - Compare: Implied vols in portfolio vs. market vols
  - Find: Using 30-day average vol, but should use EWMA
  
11:00 AM: Escalate to model team
  - Recommendation: Switch to EWMA vol calculation
  - Expected impact: Reduce exceptions to <5%
  
14:00 PM: Approve and schedule change
  - Implement next week
  - Re-backtest from historical data
  - Validate improvement before moving to production
```

#### Journey 3: Regulatory Submission (Compliance Team)

```
Quarter-end:
  - Compile 3-month backtesting results
  - Exception rates: Q1=4.2%, Q2=3.8%, Q3=4.5% (all Green ✅)
  - No exceptions in Red zone
  - All Yellow zone exceptions resolved within week
  
Generate regulatory report:
  - Narrative: Model validation framework
  - Data: Exception rates by segment
  - Evidence: Root cause analysis for top exceptions
  - Sign-offs: Risk Officer, Model Governance, CRO
  
Submit to Banco de España
  - Validates IMM approach
  - Justifies capital calculation
  - Demonstrates ongoing model governance
```

### Traffic Light System

**Status thresholds:**

| Zone | Exception Rate | Action | Escalation |
|------|---|---|---|
| **Green ✅** | 0% - 5% | Monitor, standard reporting | None |
| **Yellow ⚠️** | 5% - 8% | Investigate, root cause analysis, plan correction | To Model Governance |
| **Red 🔴** | > 8% | STOP using model for capital calc, immediate review | To CRO, Regulators |

**Key metrics to monitor:**

| KPI | Green | Yellow | Red | Owner |
|-----|-------|--------|-----|-------|
| Exception Rate | <5% | 5-8% | >8% | Risk Officer |
| Concentration (Top 5) | <40% | 40-60% | >60% | Model Governance |
| Avg Excess Size | <15% | 15-25% | >25% | Risk Officer |
| Pred/Real Correlation | >0.80 | 0.60-0.80 | <0.60 | Model Team |

### Workflow Triggers

**Automatic Actions:**

1. **Yellow Zone Entry**
   - Send alert to Risk Officer + Model Governance
   - Flag dashboard for review
   - Create incident ticket
   - Schedule investigation meeting

2. **Red Zone Entry**
   - URGENT alert to CRO
   - Escalate to Regulators
   - PAUSE capital calculation using this model
   - Emergency model review meeting

3. **Consecutive Green Zones (3+ months)**
   - Confidence increase for capital reporting
   - Document satisfactory validation in quarterly submission

4. **New Exception Type**
   - Classify as "Investigate"
   - Route to relevant team (rates, FX, equity, etc.)
   - Track until resolution

## Acceptance Criteria

**AC-PROD-BT-01**: Backtesting debe ejecutarse automáticamente cada viernes EOD sin intervención manual.

**AC-PROD-BT-02**: Reporte de backtesting debe estar listo para revisión el lunes 09:00 AM.

**AC-PROD-BT-03**: Cada excepción debe tener causa probable asignada automáticamente o manualmente documentada.

**AC-PROD-BT-04**: Si tasa de excepciones entra en Yellow zone, sistema debe generar alerta en < 1 hora.

**AC-PROD-BT-05**: Usuario debe poder investigar cualquier excepción individual en < 10 minutos usando tools de análisis.

**AC-PROD-BT-06**: Reportes regulatorios deben generarse en < 4 horas con datos de 3 meses de backtesting.

**AC-PROD-BT-07**: Dashboard debe mostrar trending de tasa de excepciones con granularidad mínima de 1 semana.

## Evidence

- C102: Backtesting v1.0 - requisitos de validación
- P036: Generador Backtesting - especificaciones
- Basilea III - Model validation framework
- BBVA Risk Policy - IMM governance

## Related Specs

- `DOM-BT-001`: Backtesting - definición y alcance
- `DOM-BT-002`: Backtesting - metodología de cálculo
- `ARCH-BT-001`: Backtesting Generator - implementación
- `PROD-IMM-001`: IMM - capacidades de producto
