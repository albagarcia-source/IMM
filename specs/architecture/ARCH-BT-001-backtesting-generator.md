---
id: ARCH-BT-001
title: "Backtesting Generator: Architecture and Implementation"
type: knowledge
layer: architecture
status: active
confidence: high
version: 1.0.0
owner: platform-team
domain: backtesting
subdomain: system-implementation
tags: [architecture, backtesting, generator, P036]
uses-data-from: [DOM-BT-001, DOM-BT-002, ARCH-IMM-001]
constrained-by: []
supersedes: []
---

## Intent

Describir la arquitectura técnica del Generador de Backtesting de BBVA: componentes del sistema, flujos de procesamiento, y cómo se integra con el motor IMM para producir reportes de validación.

## Definition

### Componentes del Generador Backtesting

**1. Historical Data Retriever**
- Extrae exposiciones predichas históricas del IMM Engine (archivo o DB)
- Consolida market data histórica de Bloomberg
- Recupera datos de posiciones cerradas (settlement info)
- Período: Últimos 24 meses (configurable)

**2. Exposure Calculator**
- Revalúa portafolios históricos con datos de mercado del día
- Aplica CSA (Credit Support Annex) vigente
- Calcula exposición realizada (mark-to-market)
- Segrega por contrapartida, tipo, región

**3. Comparison Engine**
- Compara EE predicho (de hace 1 mes) vs. EE realizado (hoy)
- Identifica excepciones (realizado > predicho)
- Calcula tasa de excepciones
- Genera matriz de análisis

**4. Analysis Module**
- Clasifica excepciones por causa (market stress, vol jump, correlation change, model weakness)
- Genera gráficos de distribución de excepciones
- Identifica contrapartidas o productos problemáticos
- Compara contra thresholds regulatorios

**5. Reporting Engine**
- Genera reportes en Excel, PDF
- Crea dashboards interactivos
- Exporta datos a Risk Data Warehouse
- Comunica resultados a Riesgos

### Flujo de Procesamiento

**Fase 1: Preparación de Datos (Semanal)**
```
1. Extraer últimos 24 meses de EE predichos del IMM Engine
2. Descargar market data histórica (Bloomberg)
3. Validar integridad de datos
4. Consolidar en staging tables
```

**Fase 2: Cálculo de Exposición Realizada (Diario)**
```
1. Para cada fecha de observación t:
   a. Obtener posiciones de portafolio en t-k (k=30 días típico)
   b. Obtener market data EOD de fecha t
   c. Revaluar cada instrumento en portafolio
   d. Aplicar CSA y netting rules
   e. Calcular exposición neta por contrapartida
2. Almacenar resultados en tabla: realized_exposure
```

**Fase 3: Comparación y Análisis (Semanal)**
```
1. Para cada fecha observada:
   a. Obtener EE predicho (de hace 30 días)
   b. Obtener EE realizado (calculado en Fase 2)
   c. Comparar: ¿EE_realizado > EE_predicho?
   d. Si sí → registrar como excepción
2. Calcular tasa de excepciones por:
   - Período (mensual, trimestral, anual)
   - Tipo de contrapartida
   - Región
   - Moneda
3. Comparar contra thresholds:
   - Aceptable: <5%
   - Atención: 5%-8%
   - Crítico: >8%
```

**Fase 4: Generación de Reportes (Semanal)**
```
1. Crear resumen ejecutivo
2. Incluir gráficos de:
   - Tasa de excepciones en tiempo
   - Distribución por tipo contrapartida
   - Top 10 contrapartidas con excepciones
   - Análisis de causas
3. Generar tabla detallada de excepciones
4. Exportar a Excel con validaciones
5. Enviar a equipo de Riesgos
```

### KPIs y Thresholds

| KPI | Green | Yellow | Red | Acción |
|-----|-------|--------|-----|--------|
| Tasa Excepciones Anual | <5% | 5%-8% | >8% | Monitor / Investigate / Adjust Model |
| Concentración Top-5 Contrapartidas | <40% | 40%-60% | >60% | Review CSA / Netting |
| Excesos Promedio (cuando excepción) | <15% | 15%-25% | >25% | Recalibrate Model |
| Correlación Pred/Real (scatter plot) | >0.80 | 0.60-0.80 | <0.60 | Model Review |

## Acceptance Criteria

**AC-BT-ARCH-01**: El backtesting debe ejecutarse de forma automática al menos semanalmente.

**AC-BT-ARCH-02**: Los datos de input deben validarse antes del cálculo (chequeos de integridad, rango, completitud).

**AC-BT-ARCH-03**: El resultado debe incluir análisis de excepciones con causa probable asignada.

**AC-BT-ARCH-04**: Si tasa de excepciones entra en Yellow o Red Zone, debe alertarse a Riesgos automáticamente.

**AC-BT-ARCH-05**: Todo cálculo debe ser auditable: registrar inputs, fórmulas aplicadas, outputs.

**AC-BT-ARCH-06**: La performance debe permitir procesar 5,000+ contrapartidas en <4 horas.

## Evidence

- P036: Generador Backtesting - especificaciones técnicas
- C102: Backtesting v1.0 - metodología y criterios de aceptación

## Related Specs

- `DOM-BT-001`: Backtesting - definición
- `DOM-BT-002`: Backtesting - metodología de cálculo
- `DOM-IMM-001`: Internal Model Method
- `ARCH-IMM-001`: IMM Module Integration
