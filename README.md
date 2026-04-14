# MAOS-ECS

Open-source Environmental Control System (ECS) development for MAOS aircraft concepts.

## Status

Concept and architecture phase.

## Safety Notice

This repository is for research and experimental development only. It is not approved for manned flight and must not be used as-is in safety-critical operation without formal system safety assessment, verification, validation, and regulatory compliance.

This project targets the Experimental Amateur-Built category. FAA certification is not a current design constraint, but engineering decisions should still follow established aerospace and safety-critical best practices where practical.

## Role in MAOS Multi-Project Architecture

This repository owns the environmental control domain for MAOS.

- Cabin thermal management (cooling, heating, distribution).
- Pressurization development path (structural provisions first, controlled pressure capability later).
- ECS controls, sensing, and subsystem-level verification.

Related repositories:

- MAOS-DESIGN owns aircraft-level packaging and structural integration baselines.
- MAOS-FCS owns flight control logic and flight-critical control paths.
- aerocommons is the website and program-level communications hub.

## Mission

Design an ECS architecture that is practical for amateur construction while maintaining disciplined systems engineering:

- Deliver useful climate control from first flight configuration.
- Preserve a clean upgrade path to pressurization without major airframe redesign.
- Use explicit interface contracts for power, ducts, controls, sensors, and fault responses.
- Build verification evidence progressively from simulation and bench tests to integrated testing.

## Scope

In scope:

- Thermal load estimation and operating envelopes.
- System architecture trade studies (vapor cycle, heat pump, bleedless approaches as applicable).
- Cabin distribution, ducting concepts, and controllability.
- Pressurization concept development, control logic, and safety monitoring.
- ECS controller firmware/software requirements and interfaces.
- Test plans, bench procedures, and data artifacts.

Out of scope:

- Aircraft primary flight controls (owned by MAOS-FCS).
- Airframe geometry and configuration governance (owned by MAOS-DESIGN).

## Interface-First Rules

Before detailed implementation, freeze and version:

- Electrical interfaces: bus voltage ranges, inrush expectations, steady-state power budgets.
- Mechanical interfaces: mounting envelopes, service clearances, mass properties.
- Fluid/air interfaces: duct diameters, pressure-drop assumptions, condensate handling.
- Control interfaces: commands, telemetry, rates, units, and fault flags.
- Fault semantics: degraded modes, shutoff triggers, and crew alert conditions.

Starter template: `INTERFACE_CONTROL_DOCUMENT_TEMPLATE.md`

## Suggested Repository Layout

- docs  Architecture notes, requirements, and verification plans
- configs  ECS operating profiles and tuning baselines
- firmware  Embedded/controller software
- sim  Thermal and pressure-domain models
- tests  Unit, integration, and bench-test procedures
- tools  Analysis scripts and log utilities

## Verification Strategy

- Requirements-to-test traceability for all critical ECS behaviors.
- Simulation-in-the-loop for thermal and pressure control logic.
- Bench validation for sensing chains, actuators, and fault handling.
- Controlled integration testing with representative aircraft interfaces.
- Structured issue tracking and regression evidence for each design change.

## Contribution Expectations

- Keep assumptions explicit and tied to data where possible.
- Separate confirmed results from hypotheses in docs and issues.
- Prefer small, reviewable pull requests with clear test evidence.
- Do not claim certification, compliance approval, or airworthiness status.

## References (Best-Practice Guidance)

Use standards and guidance documents as references only, not as claims of compliance:

- ARP4754A (system development)
- ARP4761 (safety assessment)
- DO-160 (environmental considerations)
- DO-178C / DO-254 (software/hardware process references when applicable)

## Current Milestones (As of 2026-04-14)

Current repository maturity is bootstrap-level (README-only baseline).

Near-term milestones:

1. Establish baseline repository structure (`docs`, `configs`, `tests`, `tools`, and implementation roots as needed).
2. Publish ECS top-level requirements and assumptions document.
3. Create first interface control draft for ECS electrical, mechanical, and control interfaces.
4. Define initial verification matrix for thermal control and pressurization development path.
5. Stand up first simulation scaffold for thermal-load and cabin-conditioning trade studies.

## Knowledge Migration

- Article-derived subsystem migration notes: `docs/ARTICLE_KNOWLEDGE_MIGRATION_2026Q2.md`

## Licensing

This repository uses a dual-license model:

- Source code: PolyForm Noncommercial 1.0.0 (`LICENSE-CODE`)
- Documentation and non-code design content: CC BY-NC-SA 4.0 (`LICENSE-DOCS`)

Commercial use is not granted by default. For commercial licensing, contact `contact@aerocommons.org`.

Contribution and file classification guidance: `CONTRIBUTING.md`

## Program Context

MAOS is an open-source experimental aircraft development effort. The aircraft has not yet flown. All performance values and design characteristics are targets subject to revision as evidence accumulates.
