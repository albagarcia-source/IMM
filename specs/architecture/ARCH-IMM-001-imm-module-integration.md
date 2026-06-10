---
id: ARCH-IMM-001
title: "IMM Module Integration: Data Flows and System Architecture"
type: knowledge
layer: architecture
status: active
confidence: medium
version: 1.0.0
owner: platform-team
domain: internal-model-method
subdomain: system-architecture
tags: [architecture, integration, IMM, data-flows]
uses-data-from: [DOM-IMM-001]
constrained-by: []
supersedes: []
---

## Intent

Describir la arquitectura técnica del módulo IMM en BBVA: cómo se integra con sistemas upstream (mercados, transacciones), cómo fluyen los datos, y qué componentes son responsables de cada cálculo.

## Definition

### Componentes Principales

**1. Data Consolidation**
- Extrae posiciones de TMS (Trading Management System)
- Descarga market data de Bloomberg/Reuters
- Consulta datos maestros de contrapartidas de CRM
- Consolida en formato estándar para cálculos

**2. Portfolio Aggregation**
- Agrupa transacciones por contrapartida
- Aplica netting agreements (CSA)
- Calcula exposición neta por contrapartida

**3. Monte Carlo Engine**
- Simula trayectorias futuras de precios (rates, FX, equity, commodity)
- Usa volatilidades y correlaciones de mercado
- Genera mínimo 1,000 trayectorias por escenario

**4. EE Calculation**
- Revalúa portafolio en cada nodo de simulación
- Calcula exposición esperada (EE)
- Calcula percentiles (PFE 95%, PFE 99%)

**5. Risk Reporting**
- Agrega resultados por contrapartida, tipo, región
- Calcula CVA (Credit Valuation Adjustment)
- Genera reportes de capital (SA-CCR)

### Flujos de Datos Clave

**Flujo 1: Ingesta Diaria de Posiciones**
```
TMS (EOD) → Data Consolidation → Portfolio DB → IMM Engine
```
Frecuencia: Diaria (EOD)
Latencia Máxima: 4 horas post-cierre

**Flujo 2: Actualización de Market Data**
```
Bloomberg → Data Consolidation → Market Data Cache → IMM Engine
```
Frecuencia: Intraday (mínimo 3x/día)
Latencia Máxima: 30 minutos

**Flujo 3: Cálculo de Exposición**
```
Portfolio DB + Market Data → Simulation Engine → EE Output
```
Frecuencia: EOD
Duración: 2-6 horas según volumen

### Interfaces Técnicas

**Input Interfaces**
- TMS API: Extrae posiciones con status = ACTIVE
- CRM API: Consulta datos de contrapartida (rating, CSA, netting rules)
- Bloomberg Feed: Market data en tiempo real
- Config DB: Parámetros de modelo (volatilities, correlations)

**Output Interfaces**
- Risk Data Warehouse: Exposiciones por contrapartida
- CVA System: Valuation adjustments
- Capital Calculation Engine: Inputs para SA-CCR
- Reporting API: Consultas ad-hoc de exposición

## Acceptance Criteria

**AC-ARCH-01**: El flujo de datos debe completarse en máximo 6 horas desde EOD.

**AC-ARCH-02**: La disponibilidad del IMM Engine debe ser ≥ 99.5% (max 3.6 hours downtime/mes).

**AC-ARCH-03**: Todo cambio en posiciones debe reflejarse en EE en la siguiente ejecución.

**AC-ARCH-04**: Los cálculos deben ser replicables: mismos inputs → mismos outputs.

**AC-ARCH-05**: Se debe mantener audit trail de todos los inputs y outputs.

## Evidence

- P036: Arquitectura del Generador Backtesting
- Documento metodológico: "Internal Model Method for Counterparty Credit Risk BBVA - Methodology"

## Related Specs

- `DOM-IMM-001`: Internal Model Method - conceptos
- `ARCH-BT-001`: Generador backtesting - arquitectura
