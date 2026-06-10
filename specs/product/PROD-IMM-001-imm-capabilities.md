---
id: PROD-IMM-001
title: "Internal Model Method: Product Capabilities and Scope"
type: knowledge
layer: product
status: active
confidence: high
version: 1.0.0
owner: product-team
domain: internal-model-method
subdomain: product-capabilities
tags: [product, IMM, capabilities, features]
uses-data-from: [DOM-IMM-001]
constrained-by: []
supersedes: []
---

## Intent

Definir las capacidades del producto IMM desde la perspectiva del usuario final (Risk Manager, Trader, Risk Officer): qué puede hacer, a quién beneficia, y cuál es el value proposition.

## Definition

### Value Proposition

**Para Risk Managers:**
- ✅ Calcular exposición esperada futura (EE) con precisión regulatoria
- ✅ Identificar contrapartidas de alto riesgo en tiempo real
- ✅ Cumplir con requisitos Basilea III de capital
- ✅ Tomar decisiones informadas sobre límites y colateral

**Para Traders:**
- ✅ Entender la exposición real de sus posiciones
- ✅ Evaluar impacto de nuevas transacciones en EE
- ✅ Optimizar portafolios considerando CVA
- ✅ Acceder a exposiciones en tiempo real

**Para Ejecutivos (CFO/CRO):**
- ✅ Reportar exposición al capital regulatorio de forma precisa
- ✅ Justificar modelos internos ante reguladores
- ✅ Monitorear concentración de riesgo por contrapartida
- ✅ Validar modelo IMM mediante backtesting continuo

### Capacidades Principales

#### 1. Calculation Engine

**Qué hace:**
- Calcula Expected Exposure (EE) para cada contrapartida
- Simula 1,000+ escenarios futuros de mercado
- Revalúa portafolios en cada escenario
- Produce percentiles (95%, 99%)

**Inputs:**
- Posiciones de trading (derivados, repos, swaps, FX forwards)
- Market data (rates, volatilities, FX spots, correlations)
- Contraparty master data (CSA terms, netting rules)

**Outputs:**
- Expected Exposure por contrapartida
- Potential Future Exposure (PFE)
- Effective Expected Positive Exposure (EEPE)
- Distribution of exposures over time

**Frequency:**
- EOD (End of Day) diario
- Intraday on-demand

#### 2. Real-Time Monitoring Dashboard

**Qué hace:**
- Visualiza exposiciones por contrapartida, tipo, región
- Muestra tendencias históricas de EE
- Alertas cuando exposición excede límites
- Drilldown a nivel de transacción

**Users:**
- Risk Managers
- Trading Floor
- Risk Officers

**Metrics displayed:**
- Current EE vs. Limit
- EE trend (7d, 30d, YTD)
- Top 10 counterparties by exposure
- Concentration metrics (Herfindahl)
- Margin of safety vs. threshold

#### 3. Backtesting & Model Validation

**Qué hace:**
- Compara predicciones históricas contra realizados
- Identifica excepciones (predicted < realized)
- Clasifica por causa (market stress, model weakness, data issue)
- Genera reportes de validación regulatoria

**Users:**
- Risk Officers
- Model Governance
- Regulators

**Frequency:**
- Weekly analysis
- Monthly summary report
- Quarterly regulatory submission

#### 4. CVA (Credit Valuation Adjustment)

**Qué hace:**
- Calcula ajuste de valoración crediticia
- Incorpora probabilidad de default de contrapartida
- Genera P&L impact de cambios en CVA
- Support para cálculos de capital SA-CCR

**Outputs:**
- CVA por contrapartida
- CVA delta (sensibilidad a cambios en spread)
- CVA allocation to trades

#### 5. Reporting & Export

**Qué hace:**
- Genera reportes para risk committee
- Exporta a reguladores (Banco de España, ECB)
- Integración con Risk Data Warehouse
- API para sistemas downstream

**Report types:**
- Executive summary
- Detailed exposure by counterparty
- Backtesting results
- Capital requirement reconciliation
- Limit breach alerts

### User Journeys

#### Journey 1: Daily Risk Assessment (Risk Manager)

```
09:00 AM: Open IMM Dashboard
         └─ See current exposures vs. limits
         └─ Alert: ABCorp exposure at 95% of limit
         
09:15 AM: Drill down into ABCorp position
         └─ View constituent trades
         └─ See market data that drives EE
         
09:30 AM: Discuss with trader
         └─ Request intraday recalculation
         └─ Evaluate impact of closing 50M notional
         
10:00 AM: Make decision
         └─ Approve or restrict new trades with ABCorp
         └─ Adjust collateral requirements
```

#### Journey 2: New Trade Approval (Trader)

```
14:00 PM: Want to execute €50M IRS with NewBank
         └─ System shows: Current EE = €15M, Limit = €40M
         └─ New trade would bring EE to €22M ✅ (within limit)
         
14:05 PM: Execute trade
         
14:30 PM: EOD calc includes new trade
         └─ Updated EE visible in next dashboard refresh
```

#### Journey 3: Monthly Model Validation (Risk Officer)

```
Month-end (Day 2 of month):
  - Run backtesting analysis
  - Compare: EE_predicted (from month ago) vs. EE_realized (actual)
  - Identify exceptions
  - Classify causes
  - Generate report
  - Compare to thresholds (Green: <5%, Yellow: 5-8%, Red: >8%)
  - If Yellow/Red: trigger investigation + escalation
```

### Non-Functional Requirements

| Requirement | Target | Priority |
|-------------|--------|----------|
| **Availability** | 99.5% (max 3.6h downtime/month) | Critical |
| **EOD Completion** | < 6 hours post-market close | Critical |
| **Intraday Response** | < 30 minutes for on-demand calc | High |
| **Data Freshness** | Real-time (< 5 min lag) | High |
| **Scalability** | 5,000+ counterparties | High |
| **Auditability** | 100% traceability of inputs/outputs | Critical |
| **Security** | Role-based access control (RBAC) | Critical |
| **Disaster Recovery** | RTO < 2 hours, RPO < 1 hour | High |

## Acceptance Criteria

**AC-PROD-01**: El dashboard debe mostrar exposiciones actualizadas en menos de 5 minutos después de cada cálculo EOD.

**AC-PROD-02**: Todo usuario debe poder exportar reportes de exposición en Excel o PDF sin asistencia técnica.

**AC-PROD-03**: Las alertas de límite superado deben comunicarse dentro de 15 minutos post-detection.

**AC-PROD-04**: 95% de cálculos de EE deben completarse en tiempo < 3σ del promedio (sin outliers).

**AC-PROD-05**: Toda transacción de entrada debe ser validada antes de incluirse en cálculo (chequeos de formato, rango, completitud).

**AC-PROD-06**: El sistema debe permitir recalcular exposiciones históricas para cualquier fecha pasada en < 1 hora.

**AC-PROD-07**: Cada cálculo debe ser auditable: quién lo solicitó, cuándo, con qué inputs, qué outputs produjo.

## Evidence

- Internal Model Method BBVA Methodology Document
- Basilea III - Counterparty Credit Risk Framework
- BBVA Risk Management Policy
- User feedback from Risk Managers and Traders

## Related Specs

- `DOM-IMM-001`: Internal Model Method - conceptos técnicos
- `ARCH-IMM-001`: IMM Module Integration - implementación
- `PROD-BT-001`: Backtesting - product capabilities
