# MAOS-ECS Article Knowledge Migration (2026 Q2)

Purpose: Capture environmental control and thermal-management knowledge from aerocommons into MAOS-ECS execution artifacts.

Scope note: This is R&D guidance for Experimental Amateur-Built development. It is not a certification claim.

## Source Articles

- 2026-04-05-maos-ecs-phase1-bom.md
- 2026-04-11-naca-duct-design-gate-for-pod-ecs.md
- 2026-04-03-waste-heat-ice-protection-battery-thermal.md
- 2026-04-06-maos-geometry-design-decisions.md

## Imported Decisions and Working Baseline

- ECS Phase 1 should prioritize measurable bench and integrated test evidence over architectural breadth.
- NACA duct integration is a gated decision with explicit acceptance metrics, not a styling choice.
- Waste heat should be treated as a usable energy stream for thermal management and anti-ice concepts where practical.
- High-altitude operating context requires explicit cooling and pressure-flow validation.

## ECS Guidance to Carry Into This Repo

- Maintain a test-first decision gate for ducting and heat-exchange architecture.
- Keep thermal management coupled to propulsion and battery operating envelopes.
- Define clear subsystem-level telemetry requirements so control and safety logic can react deterministically.

## Open Decisions Assigned to MAOS-ECS

- Finalize NACA duct go/no-go criteria and measurement plan.
- Define Phase 1 ECS BOM freeze point and allowed substitutions.
- Confirm waste-heat routing strategy and integration constraints.
- Define cold-weather and icing operating assumptions for Phase 1 tests.

## Immediate Work Items

1. Create docs/NACA_DUCT_DECISION_GATE_V1.md with pass/fail metrics.
2. Create docs/ECS_PHASE1_BOM_BASELINE.md and change-control rules.
3. Create docs/THERMAL_ENERGY_FLOW_AND_HEAT_REUSE.md with interface points.
4. Publish ECS telemetry signal list for MAOS-FCS and MAOS-MOTOR consumers.

## Suggested Deliverables to Add Next

- docs/NACA_DUCT_DECISION_GATE_V1.md
- docs/ECS_PHASE1_BOM_BASELINE.md
- docs/THERMAL_ENERGY_FLOW_AND_HEAT_REUSE.md
- docs/ECS_TELEMETRY_AND_ALERT_SIGNALS_V0.md
