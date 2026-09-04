# MAOS LV Electrical Load List — First Pass (ASTM F2490 Load-vs-Flight-Phase Matrix)

**Status:** Draft v1 — first pass, conceptual level
**Subsystem:** SYSTEMS — electrical distribution, LV bus
**Date:** 2026-08-29
**Related issue:** AER-437
**Scope:** LV bus (28V-class) only. The HV bus (DG-004, ~400–800V-class, jointly owned with PROPULSION) is explicitly **out of scope** and unresolved on AER-73 — nothing in this document assumes or implies a value for it.

## 0. Why this exists, and why now

DG-004 (HV bus voltage) has blocked most of SYSTEMS' electrical architecture work for three weeks (see AER-132, AER-271 weekly status). The LV bus does not depend on it — 28V-class is SYSTEMS' own call, consistent with the MIL-STD-704 "in-standard" ceiling used across GA and More-Electric-Aircraft practice. This is the first published, numbers-with-a-method LV load list rather than a narrative placeholder.

## 1. Method

**ASTM F2490** ("Standard Guide for Aircraft Electrical Load and Power Source Capacity Analysis") is the method anchor: tabulate every load's steady-state and intermittent demand, evaluate across *all applicable flight phases* (not a single worst-case guess), classify each load essential/non-essential, and **re-run the matrix with a generation source degraded** to quantify redundancy claims instead of asserting them. Installed-system context and penalty framing follow Moir & Seabridge (*Aircraft Systems*) and Gudmundsson (*General Aviation Aircraft Design*).

This is a **first pass at the conceptual design stage**: most consuming subsystems (FCS, ICE management, landing gear) have not yet selected hardware, so several line items are first-principles ROM (rough order of magnitude) estimates rather than vendor-specified numbers. Every non-firm figure is tagged **[ROM]** with its derivation, and the biggest open unknowns are ranked in §7 rather than hidden inside a confident-looking total. Re-running this matrix is cheap — it should happen again each time a flagged item resolves, not once at the end.

## 2. LV bus definition (proposed — not yet a ratified gate)

- **Nominal voltage: 28V-class DC.** SYSTEMS' own domain call, independent of the open DG-004 HV gate. 28V and ±270V are the two established More-Electric-Aircraft tiers under MIL-STD-704; 28V is the appropriate tier for the load classes below (avionics, EFI, lighting, small actuation) — nothing here needs ±270V-class hardware.
- **Proposed source architecture** (shown so the degraded-bus rerun in §6 has a concrete case to test against — this is a proposal for board/PROPULSION concurrence, not an assertion of a decided architecture):
  - **Source A (primary):** a 28V DC-DC converter fed from the HV bus, live whenever the ICE/generator set is running. Series-hybrid architectures routinely derive LV power this way rather than carrying a second prime mover just for LV.
  - **Source B (buffer):** a dedicated LV battery, sized for (a) a stated reserve duration on essential loads alone if Source A is lost, and (b) whatever momentary engine-start current the LV bus must supply (see LV-17, §7 item 2).
  - This LV-side pattern mirrors the standing HV battery reserve doctrine (`maos-propulsion-redundancy-battery`, 40 kWh / 30 min at the HV level) at LV scale, for architectural consistency — proposed here, not yet a board-ratified target.

## 3. Flight phases

Ground/Preflight → Engine Start → Taxi → Takeoff/Climb → Cruise → Descent/Approach → Landing/Rollout → Post-flight/Shutdown. Standard ASTM-style phase set; sufficient resolution for a first pass without a published mission-duration profile.

## 4. Load inventory

All currents are bus-side at 28V-class nominal. "Essential" = required for continued safe flight and landing; "Non-essential" = situational-awareness or comfort, sheddable under a degraded source. Data provenance is tagged per line: **[MFR]** manufacturer-published spec, **[COMM]** community-measured/forum-sourced figure (real hardware, secondary source), **[ROM]** first-principles engineering estimate with no vendor data yet.

### Avionics / display

| ID | Load | Tier | Steady power | Peak/intermittent | Mass (ROM) | Phases | Source |
|---|---|---|---|---|---|---|---|
| LV-01 | pyEfis PFD display (RPi4 + 7" touchscreen) | Essential | 12 W (0.43 A) | 15 W (0.54 A) | 0.5 kg | All | [COMM] RPi4+peripherals class, crewdogelectronics/rtl-sdr.com Stratux-hardware measurements |
| LV-02 | pyEfis MFD display (2nd panel) | Non-essential | 12 W (0.43 A) | 15 W (0.54 A) | 0.5 kg | All | same basis |
| LV-03 | Hub Pi (CAN-FIX↔Ethernet bridge + recorder, roadmap §7) | Non-essential | 7 W (0.25 A) | 10 W (0.36 A) | 0.3 kg | All | [ROM] RPi4 headless, lighter than Stratux (no SDR/GPS) |
| LV-04 | Stratux (GPS, ADS-B In, AHRS) | Non-essential† | 12.5 W (0.45 A) | 15 W (0.54 A) | 0.35 kg | All | [COMM] rtl-sdr.com / crewdogelectronics.com Pi4+2×SDR+GPS measurements |
| LV-05 | OnSpeed-Gen2 (AOA, pitot-static, AHRS) | Essential | 3 W (0.11 A) | — | 0.2 kg | All | [ROM] Arduino/embedded-class compute |
| LV-06 | GNX-375 (certified IFR GPS nav, ADS-B Out, Mode S) | Essential | 17 W typ (0.6 A) | 25 W max (0.9 A) | 0.7 kg | All | [MFR] Garmin-published 1.20 A typ / 1.80 A max at 14V, halved for 28V-class at equal power |
| LV-07 | MGL V16 COM radio | Essential | 4 W RX cont. (0.15 A) | 35 W TX burst (1.25 A), ~10% duty in cruise | 0.3 kg | All | [MFR] MGL datasheet: 0.3 A RX / 2.5 A TX at 13.8V |
| LV-08 | CAN-FIX "Smart Puck" magnetometer | — | ~1.5 W (0.05 A) | — | 0.1 kg | future | [ROM] — **not yet built** (roadmap: "the gap"); excluded from all totals below, listed for completeness only |
| LV-09 | Avionics bus infrastructure (relays, fusing, panel backlighting/switches) | Essential | 0.8 A lump | — | 0.3 kg | All | [ROM] first-principles lump |

† Stratux is tiered non-essential because it sits in a genuinely redundant position: GPS position is also carried by the certified GNX-375, and AHRS attitude is also carried by OnSpeed. Losing Stratux loses ADS-B traffic/weather situational awareness, not a primary flight parameter. This redundancy is exercised, not just claimed, in §6.

### Flight control system (triplex FBW)

| ID | Load | Tier | Steady power | Peak/intermittent | Mass (ROM) | Phases | Source |
|---|---|---|---|---|---|---|---|
| LV-10 | FCC lane computer ×3 (STM32H7-class + dual CAN-FD) | Essential | 15 W group (0.54 A) | — | 0.6 kg group | All | [ROM] — MAOS-FCS `hardware_v0_1.md` is still a draft target with no bench current data |
| LV-11 | Air data computer ×2 | Essential | 4 W group (0.14 A) | — | 0.2 kg group | All | [ROM] |
| LV-12 | Primary flight-control EMAs, 4 axes (pitch/roll/yaw/spoiler) | Essential | **Cruise avg ≈9–12 A group; climb/landing maneuvering avg ≈15–20 A group** | Per-axis peak ≈20–25 A (non-concurrent; diversity ≈0.6–0.7 applied to group figures above) | **TBD — likely 1–3 kg/axis, 4–12 kg group** | All, magnitude varies by phase | [ROM] — **no actuator vendor selected; see §7 item 1, this is the single largest open number in the whole document** |

### ICE management (LV side only — the HV generator/motor stays out of scope)

| ID | Load | Tier | Steady power | Peak/intermittent | Mass (ROM) | Phases | Source |
|---|---|---|---|---|---|---|---|
| LV-13 | ECU (candidate range: Speeduino / MS3-Pro / SDS EM-6) | Essential | 8–12 W (0.3–0.4 A) | — | 0.3–0.5 kg | Start→Post-flight | [ROM]+[COMM] reference point: MS2(V3) core measured 0.77 A at 12 V unloaded (MSEXTRA forum); vendor not yet selected (`ENGINE_MGMT_CANDIDATE_MATRIX_V1.md`, weighted-score leader Speeduino 3.55, not ratified) |
| LV-14 | Ignition coils ×4 (coil-on-plug) | Essential | avg ≈4–8 A group | peak ≈10 A (single-coil dwell pulse, firing order keeps them non-overlapping) | 0.6 kg group | Start→Post-flight | [COMM] automotive COP dwell-current forum data (LS2/D585-class coils, 7–11 A saturated) |
| LV-15 | Fuel injectors ×4 | Essential | avg ≈2–4 A group | — | 0.2 kg group | Start→Post-flight | [ROM] standard PWM-driven EFI injector duty |
| LV-16 | Fuel boost/lift pump (turbo-EFI class) | Essential | ≈190 W → ≈7–8 A @28V-class | — | 0.4 kg | Start→Post-flight (continuous whenever engine runs) | [COMM] Walbro 450-class: 14.1 A at 13.5V manufacturer rating, 15–21 A real-world under boost pressure. **Flag: sourced pump hardware is 12V-class automotive; a 28V-native pump or a dedicated DC-DC step-down for this one high-current item is still open.** |
| LV-17 | ICE starter | Essential (Engine Start phase only) | **UNRESOLVED: 0 A if the HV generator motors the engine for start; 150–300 A momentary (≈2–4 kW peak, 2–5 s) if a dedicated LV starter is required** | — | TBD | Engine Start only | [ROM] motorcycle-class starter precedent. **Flag: see §7 item 2 — this is a live architecture question, not a sizing detail.** |

### Environmental (LV portion only)

| ID | Load | Tier | Steady power | Peak | Mass (ROM) | Phases | Source |
|---|---|---|---|---|---|---|---|
| LV-18 | Cabin blower/vent fan | Non-essential | 50–80 W (2–3 A) | — | 0.4 kg | Taxi→Post-flight | [ROM] automotive HVAC blower-motor class |

Cabin heating/cooling (compressor or heater core, if a vapor-cycle or other active thermal system is selected) is **excluded from this LV pass** — see §7 item 6.

### Landing gear (contingent on GEAR's open concept decision)

| ID | Load | Tier | Steady power | Peak | Mass (ROM) | Phases | Source |
|---|---|---|---|---|---|---|---|
| LV-19 | Gear actuators ×2 (electric linear, only if a retract concept is selected) | Essential when installed | — | 10–15 A each @28V-class, transient only, few seconds | TBD | Takeoff/Climb (retract) + Descent/Approach (extend) | [ROM]. **Flag: contingent — `GEAR_CONCEPT_TRADE_STUDY_V1.md` is still "Working Draft," Concept E (electric retract) is the recommendation but not selected; Concept A (fixed gear) would zero this line entirely.** |

### Lighting

| ID | Load | Tier | Steady/avg | Peak | Mass (ROM) | Phases | Source |
|---|---|---|---|---|---|---|---|
| LV-20 | Nav lights ×3 (LED) | Essential (night)/Non-essential (day VMC) | 1.5 A group | — | 0.15 kg | All (night case) | [MFR] AeroLED-class 0.5 A/light |
| LV-21 | Strobe lights ×2–3 (LED) | Essential (collision avoidance, day+night) | avg ≈3–6 A group (low duty-cycle flash) | peak ≈10 A group ganged (breaker sizing) | 0.2 kg | All airborne phases | [MFR] 3.5 A peak/light |
| LV-22 | Landing light ×1–2 | Non-essential (recommended) | 4–8 A group | — | 0.3 kg | Takeoff/Climb, Descent/Approach, Landing/Rollout | [MFR] ≈4 A/light |
| LV-23 | Taxi light | Non-essential | 4 A | — | 0.15 kg | Taxi, Landing/Rollout | [MFR] |

### Ice-protection-adjacent (baseline only — NOT the wing/prop method decision)

| ID | Load | Tier | Steady power | Peak | Mass (ROM) | Phases | Source |
|---|---|---|---|---|---|---|---|
| LV-24 | Pitot/AOA probe heat | Essential, icing conditions only (abnormal case) | 30–50 W (1–2 A) | — | 0.1 kg | Any phase, icing column only | [ROM]. **This is the baseline probe heater common to virtually every wing/prop ice-protection method — it is not a stand-in for that decision, which stays open per the DG-level ice-protection method trade (fluid/TKS vs. electrothermal vs. bleed-air, still unresolved and not sized here).** |

## 5. Load-vs-flight-phase matrix (normal bus, night-ops case)

All figures in amps at 28V-class, steady/average basis (transients called out separately, not folded into the sustained total).

| Phase | Essential (A) | Non-essential (A) | Total (A) | Flagged transients not included above |
|---|---|---|---|---|
| Ground/Preflight | 6.6 | 2.4 | 9.0 | — |
| Engine Start | 23.1 | 2.4 | 25.5 | **+ starter: 0–300 A momentary (LV-17)** |
| Taxi | 24.1 | 7.9 | 32.0 | — |
| Takeoff/Climb | 43.6 | 9.9 | 53.5 | **+ gear retract: 0–30 A, few sec (LV-19)** |
| Cruise | 34.6 | 3.9 | 38.5 | — |
| Descent/Approach | 40.6 | 9.9 | 50.5 | **+ gear extend: 0–30 A, few sec (LV-19)** |
| Landing/Rollout | 40.6 | 13.9 | 54.5 | — |
| Post-flight/Shutdown | 3.1 | 1.4 | 4.5 | — |

Day-VMC case: subtract 1.5 A (LV-20 nav lights) from each essential column where flight is day-VFR; strobes are retained as essential in both cases per standard collision-avoidance practice.

**Reading this table:** the engine-dependent group (LV-13 through LV-16 — ECU, ignition, injectors, fuel pump ≈17 A) and the FCS actuator group (LV-12, 9–20 A depending on phase) are the two largest sustained contributors, and neither is a firm number yet (§7). The steady-state totals above are defensible; the bracketed transients are not sized because the two things that would size them (starter architecture, gear concept) are open elsewhere in the org.

## 6. Degraded-bus rerun — Source A (DC-DC converter) lost, Source B (battery) alone

Per ASTM F2490 practice, redundancy is quantified by re-running the matrix with a source removed, not asserted. With Source A (HV-fed DC-DC converter) down, the LV bus runs on the battery buffer alone. Non-essential loads (LV-02, LV-03, LV-04, LV-18, LV-22, LV-23) are shed; only the essential column from §5 remains.

**Worst sustained essential draw (excluding momentary transients): Takeoff/Climb, 43.6 A.**

Proposed reserve target (for board concurrence — chosen for consistency with the standing HV battery reserve doctrine, not yet a ratified requirement): **30 minutes at worst-case sustained essential load.**

- Energy required: 43.6 A × 0.5 h ≈ **21.8 Ah useable**
- With a 20–30% depth-of-discharge margin (voltage-sag and cycle-life headroom, standard LiFePO₄ practice): **≈27–30 Ah nameplate**
- At LiFePO₄ energy density ROM (~100–120 Wh/kg): 28V nominal × 30 Ah ≈ 840 Wh → **mass ROM ≈7–8.4 kg**

**This mass is published to Structures as the LV battery placeholder (§8) — it is a ROM pending the reserve-duration target being formally set, not a component selection.**

**Coupling flag:** if LV-17 (ICE starter) resolves toward a dedicated LV starter rather than HV-motor start, this same battery must also deliver 150–300 A momentary crank current — a discharge-rate (C-rate) requirement layered on top of the Ah sizing above, potentially forcing a different cell chemistry/format than the Ah-only sizing would suggest. Resolving LV-17 before finalizing the LV battery spec avoids re-sizing it twice.

## 7. Open items, ranked by how much they could move the numbers above

1. **FCS primary-actuator power (LV-12) — dominant unknown.** No actuator vendor selected; the ROM band (9–20 A avg, group mass 4–12 kg) is the single largest swing factor in this entire load list. Needs FCS actuator hardware selection. No FCS agent is currently staffed (MAOS-FCS is dormant this cycle per org STATE.md) — this stays flagged rather than chased, and should be revisited the moment FCS activates.
2. **ICE starter architecture (LV-17) — binary, not incremental.** Does the HV generator motor the ICE for start (eliminating a dedicated LV starter and its 150–300 A momentary draw entirely), or is a separate LV starter required? This is a real architecture question for PROPULSION/SYSTEMS jointly, not asserted either way here. It also directly sizes the LV battery's discharge-rate requirement (§6).
3. **ECU vendor selection (LV-13).** Affects LV ECU power by roughly 2× across the three screened candidates (`ENGINE_MGMT_CANDIDATE_MATRIX_V1.md`); the weighted-score leader (Speeduino) is not yet ratified.
4. **Landing gear retract decision (LV-19).** `GEAR_CONCEPT_TRADE_STUDY_V1.md` recommends Concept E (electric retract) but is still "Working Draft." Concept A (fixed gear) zeros this line entirely. No GEAR agent is currently staffed.
5. **LV battery reserve-duration target (§6).** The 30-minute figure proposed here is for board concurrence, chosen only for consistency with the existing HV reserve doctrine — it has not been set as a requirement.
6. **ECS compressor/heater — excluded from this pass.** Active cabin conditioning beyond the LV-18 blower (if a vapor-cycle or other compressor-based system is selected) is very likely a candidate for HV-class power (typical compressor draw 500 W–1 kW+) rather than LV. That sizing is a separate MAOS-ECS task, not fabricated here.
7. **Wing/prop ice-protection method — stays open.** LV-24 (pitot/AOA probe heat) is a baseline safety item included regardless of which wing/prop method is eventually chosen; it is not a proxy for that decision, which remains a live fluid-vs-thermal trade per the ice-protection memory.
8. **Magnetometer/Smart Puck (LV-08) — not yet built.** Excluded from all totals; listed so it isn't forgotten once it exists.

## 8. Interfaces published

- **To PROPULSION:** proposed LV bus architecture (§2 — DC-DC converter off the HV bus + LV battery buffer), 28V-class nominal voltage, and the steady-state/peak current envelope in §5–§6. The LV starter question (§7 item 2) is a joint open item.
- **To Structures:** LV battery mass ROM (≈7–8.4 kg, §6, pending reserve-duration ratification), and the avionics/FCC-lane-computer mass/volume placeholders in §4 for packaging.
- **To Safety:** the essential-load list (§4 tier column) and the Source-A-lost degraded case (§6) as FHA input for the LV bus single point of failure at the DC-DC converter.
- **To MAOS-ECS:** LV-18 (cabin blower) is included here; the compressor/heater exclusion (§7 item 6) is flagged for ECS's own sizing pass.

## 9. Confidence and fidelity boundary

This is a conceptual-level first pass, not a detailed design ELA. Confidence is highest on COTS avionics with manufacturer or community-measured figures (LV-01 through LV-09, LV-20 through LV-23) and lowest on anything with no hardware selected yet (LV-12 FCS actuators, LV-17 starter, LV-19 gear). Treat every [ROM]-tagged figure as a placeholder to be replaced, not a committed budget line, and re-run this matrix each time one of the §7 items resolves rather than waiting for all of them.
