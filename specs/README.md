# Knowledge-Driven Development (KDD) Specs Repository

## Estructura

Este repositorio contiene las especificaciones técnicas y de negocio del dominio **Internal Model Method (IMM)** y **Backtesting** de BBVA, organizadas según la metodología KDD.

### Carpetas

- **`domain/`** — Especificaciones de reglas de negocio y dominio
  - `DOM-IMM-*`: Conceptos y metodología del Internal Model Method
  - `DOM-BT-*`: Definición y cálculo del backtesting

- **`architecture/`** — Especificaciones de arquitectura técnica
  - `ARCH-IMM-*`: Integración e implementación del módulo IMM
  - `ARCH-BT-*`: Generador de backtesting

- **`governance/`** — Decisiones arquitectónicas y RFCs (próximas)

### Convención de Nomenclatura

Cada especificación sigue el patrón:
```
{LAYER}-{DOMAIN}-{NUMBER}-{slug}.md
```

Ejemplos:
- `DOM-IMM-001-internal-model-method.md`
- `ARCH-BT-001-backtesting-generator.md`

### Relaciones entre Specs

```
DOM-IMM-001 (qué es IMM)
    ↓
    ├→ ARCH-IMM-001 (cómo implementar IMM)
    └→ DOM-BT-001 (para qué hacer backtesting)
        ↓
        ├→ DOM-BT-002 (cómo calcular backtesting)
        └→ ARCH-BT-001 (arquitectura del generador)
```

### Cómo usar estas specs

1. **Para entender el dominio**: Leer specs en `domain/`
2. **Para implementar**: Leer specs en `architecture/`
3. **Para validar**: Usar acceptance criteria de cada spec
4. **Para generar artefactos**: Usar campos `activates` en work specs

### Estados de una Spec

- **draft**: En desarrollo, no lista para producción
- **active**: Validada, en uso
- **deprecated**: Reemplazada por otra spec
- **archived**: Histórica, no aplica

### Próximos pasos

- [ ] Crear specs de **producto** (PROD-*) con journeys de usuario
- [ ] Crear specs de **features** (FEAT-*) con requisitos detallados
- [ ] Crear **governance docs** (RFC, ADR) para decisiones arquitectónicas
- [ ] Crear **work specs** (WRK-SPEC) para sprints de desarrollo
- [ ] Establecer ciclo de **consolidación** para incorporar learnings

---

**Última actualización**: 2026-06-10
**Mantener por**: Risk Platform Team
