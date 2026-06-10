---
id: PROD-DLV-001
title: "Data & Collateral Management: Product Capabilities"
type: knowledge
layer: product
status: active
confidence: high
version: 1.0.0
owner: product-team
domain: risk-management
subdomain: data-and-collateral
tags: [product, data-management, collateral, CSA, market-data]
uses-data-from: [DOM-IMM-001, DOM-BT-001]
constrained-by: []
supersedes: []
---

## Intent

Definir las capacidades de gestión de datos y colateral que soportan los cálculos IMM y Backtesting: cómo se integran datos, cómo se mantienen maestros, y cómo se aplican reglas CSA.

## Definition

### Value Proposition

**Para Risk Managers:**
- ✅ Datos de mercado actualizados en tiempo real
- ✅ Contrapartidas con CSA/netting rules siempre vigentes
- ✅ Transparencia en qué datos se usan en cada cálculo
- ✅ Capacidad de validar datos antes de cálculos

**Para Operations:**
- ✅ Integración automática de posiciones de TMS
- ✅ Descarga automática de market data (Bloomberg, Reuters)
- ✅ Validación automática de integridad de datos
- ✅ Alertas si faltan datos o hay anomalías

**Para Traders:**
- ✅ Posiciones reflejadas en exposiciones dentro de minutos
- ✅ Claridad sobre CSA terms que aplican a cada contrapartida
- ✅ Control sobre cuándo se incluye una transacción en cálculos

### Capacidades Principales

#### 1. Position Ingestion & Management

**Qué hace:**
- Extrae posiciones de TMS (Trading Management System) en tiempo real
- Valida formato, completitud, valores razonables
- Consolida por contrapartida
- Aplica status filters (ACTIVE, PENDING, SETTLED)
- Mantiene histórico de posiciones

**Data Sources:**
- TMS (primary): Derivados, repos, forwards
- Settlement System: Confirmaciones
- Middle Office: Confirmación de contrapartidas

**Frequency:**
- Real-time feed (changes within 5 minutes)
- EOD snapshot (T+0)
- Historical archive (T-36+ months)

**Validations:**
- Notional amount: reasonable range (min-max by product)
- Counterparty valid: exists in master data
- Currency: ISO 4217 codes
- Dates: start < maturity, maturity > today
- Rates/Spreads: within market ranges

**Alerts on failure:**
- Missing notional → REJECT
- Unknown counterparty → HOLD for manual approval
- Invalid dates → REJECT
- Out-of-range spread → WARN (can proceed with approval)

#### 2. Market Data Integration

**Qué hace:**
- Descarga market data de múltiples fuentes
- Normaliza a formato estándar
- Valida y detecta anomalías
- Mantiene histórico para backtesting
- Distribuye a cálculos

**Data subscriptions:**
- Bloomberg terminal: Rates, FX, equity, commodity
- Reuters: Volatilities, correlations
- Internal: Implied vols from options book
- External feeds: Central banks (for OIS rates)

**Data types:**
- Spot prices (FX, equity, commodity)
- Interest rates (EONIA, EURIBOR, LIBOR, OIS)
- Volatilities (implied, historical, tenor surface)
- Correlations (equity, commodity, cross-asset)
- Spreads (credit, basis, repo)
- Dividend yields (equity)

**Frequency:**
- Intraday: 3x/day (morning, midday, EOD)
- EOD: snapshot at 17:00 CET
- Historical: 3 years of daily data

**Validation rules:**

| Data | Check | Action on Failure |
|------|-------|-------------------|
| FX Spot | Within ±5% of 30d avg | WARN, use last valid |
| Vol | >0%, <100% | REJECT, use historical avg |
| Rate | Within ±200bps of 30d avg | WARN, investigate |
| Correlation | Between -1 and +1 | REJECT |

#### 3. Counterparty Master Data

**Qué hace:**
- Mantiene registro maestro de contrapartidas
- Almacena CSA terms, netting agreements
- Tracks rating, industria, geografía
- Links a CRM, Bloomberg, Reuters IDs
- Gestiona cambios (new, updates, deactivation)

**Data attributes:**
- Identifiers: Internal ID, Bloomberg, Reuters, LEI
- Legal: Entity name, jurisdiction, consolidation status
- Risk: Credit rating, probability of default
- Geography: Home country, risk region
- CSA: Termination clause, eligible collateral, haircuts, threshold
- Netting: Legal netting agreement ID, scope, multiplier
- Limits: Credit limit, exposure limit, concentration limit
- Contacts: Credit officer, operations contact

**Change management:**
- New counterparty: Require approvals (credit, legal, ops)
- Rating change: Auto-update, trigger sensitivity analysis
- CSA termination/signature: Manual update with evidence
- Deactivation: Mark inactive, keep historical data

**Frequency:**
- Real-time updates (when data available)
- Daily refresh from CRM
- Monthly validation audit

#### 4. CSA & Netting Rules Engine

**Qué hace:**
- Aplica Credit Support Annex terms a exposiciones
- Calcula colateral requerido vs. disponible
- Implementa netting agreements
- Genera notificaciones si colateral insuficiente
- Soporta múltiples acuerdos por contrapartida

**CSA Rules:**
- Eligible collateral types (cash, bonds, equities)
- Haircuts por tipo y rating
- Thresholds (minimum transfer amount, rounding)
- Collateral cure period
- Termination event triggers

**Netting agreements:**
- Close-out netting: Todos los trades con contrapartida se nettean al default
- Payment netting: Cash flows se nettean (reduce exposición)
- Multiplier: Factor de aplicación (e.g., 80% netting efficiency)

**Workflow:**

```
Para cada contrapartida:
  1. Obtener suma de exposiciones positivas (EE)
  2. Aplicar colateral disponible según CSA
  3. Restar colateral posted by counterparty
  4. Aplicar haircuts según collateral type
  5. Resultado: Exposición neta post-colateral
  
  Si Exposición > Threshold:
    → Generar Margin Call Notice
    → Enviar a Operations/Treasury
    → Track fulfillment
```

#### 5. Data Quality & Governance

**Qué hace:**
- Monitoring continuo de integridad de datos
- Reportes de data quality scores
- Incident tracking y resolution
- Audit trail de todos los cambios
- Data lineage (de fuente a cálculo)

**Metrics:**
- Completeness: % de campos populated
- Timeliness: % de datos EOD antes de cálculos
- Accuracy: % de validaciones passed
- Consistency: Cross-system reconciliation

**Governance board:**
- Data owner: Quién es responsable por cada dataset
- SLA: Expectativas de disponibilidad y precisión
- Issue escalation: Cómo reportar y resolver problemas
- Change control: Cómo hacer cambios a estructuras de datos

### User Journeys

#### Journey 1: Daily Operations (Operations Officer)

```
07:00 AM: Data ingestion starts automatically
  - TMS positions: 2,500 trades
  - Bloomberg market data: rates, FX, vols
  - Status: In progress
  
07:45 AM: Operations Officer checks dashboard
  - Positions received: ✅ 2,500
  - Market data: ✅ Complete
  - Validations passed: ✅ 2,498 (2 warnings)
  
07:50 AM: Investigate warnings
  - Trade 001: JPY/USD rate 5% outside range
    → Investigation: Market noise, use 30d avg
    → Approved
  - Trade 500: Unknown counterparty ID
    → Investigation: Typo, should be ABC Corp
    → Corrected
  
08:00 AM: All green ✅
  - Data locked for calculations
  - IMM engine starts
  
14:00 PM: EOD snapshot taken
  - Final data archived for backtesting
```

#### Journey 2: New Counterparty Onboarding (Credit Officer)

```
Trade with new counterparty arrives:
  - TMS flags: "Unknown counterparty: XYZ Bank"
  - System blocks position from calculation
  - Alert to Credit Officer
  
09:30 AM: Credit Officer starts onboarding
  - Search: "XYZ Bank"
  - Not in master data → Create new
  
09:45 AM: Populate master data form
  - Legal name, LEI, domicile
  - Credit rating: A (from Bloomberg)
  - CSA: "CSA with Schedule (2018)"
    → Eligible collateral: EUR, USD cash + bonds
    → Threshold: EUR 1M
    → Haircut: 2% for bonds
  
10:15 AM: Submit for approval
  - Credit approval: ✅
  - Legal approval: ✅
  - Operations: ✅
  
10:30 AM: Counterparty active in system
  - Position now included in calculations
  - Exposed to CSA rules
  - Included in exposure monitoring
```

#### Journey 3: CSA Enforcement (Treasury)

```
Daily EOD calculation:
  - GoldmanSachs EE: EUR 150M
  - Collateral posted by GS: EUR 80M
  - Hair cut: 2%
  - Net exposure: 150 - (80 × 0.98) = 71.6M
  - CSA Threshold: EUR 100M
  
Result: No action needed (71.6M < 100M)

Next day after market stress:
  - GoldmanSachs EE: EUR 185M
  - Collateral still: EUR 80M (not yet posted)
  - Net exposure: 185 - (80 × 0.98) = 106.6M
  - CSA Threshold: EUR 100M
  
Result: MARGIN CALL
  - System generates notice: EUR 6.6M additional collateral required
  - Sends to Treasury/Legal
  - Treasury negotiates timing and settlement
  - System tracks until fulfilled
```

## Acceptance Criteria

**AC-PROD-DLV-01**: Posiciones nuevas debe estar disponibles para cálculos dentro de 5 minutos de llegada a TMS.

**AC-PROD-DLV-02**: Market data EOD debe estar completo antes de iniciar cálculos (máximo 17:30 CET).

**AC-PROD-DLV-03**: 99% de validaciones deben pasar sin intervención manual (SLA: <1% manual fixes).

**AC-PROD-DLV-04**: Toda contrapartida debe tener CSA terms definidos antes de incluirse en cálculos.

**AC-PROD-DLV-05**: Colateral disponible debe reconciliarse con Treasury daily (SLA: < EUR 1M variance).

**AC-PROD-DLV-06**: Data lineage debe ser 100% trazable: de fuente original → cálculo final.

**AC-PROD-DLV-07**: Si datos críticos faltan/fallan, sistema debe alertar en < 1 hora antes de cálculos planned.

## Evidence

- TMS Integration specifications
- Bloomberg/Reuters subscription agreements
- CSA documentation (sample agreements)
- Market data SLAs
- Data governance policy

## Related Specs

- `DOM-IMM-001`: Internal Model Method - data requirements
- `ARCH-IMM-001`: IMM Module Integration - data flows
- `PROD-IMM-001`: IMM - product capabilities
- `PROD-BT-001`: Backtesting - uses this data
