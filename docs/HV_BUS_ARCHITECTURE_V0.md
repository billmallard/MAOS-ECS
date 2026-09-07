# MAOS HV Bus Architecture — V0 (DG-004, ~700V DC Nominal)

**Status:** Draft v0 — first-pass architecture, pending final DG-004 ratification
**Subsystem:** SYSTEMS — HV electrical distribution (jointly owned with PROPULSION per DG-004)
**Date:** 2026-09-06
**Related issue:** AER-671 (child of AER-56). Concurrence basis: AER-669 (SYSTEMS review of `GENSET_PACKAGE_SPEC_V0.md`, MAOS-ICE PR #20).
**Data provenance tags** (matching `GENSET_PACKAGE_SPEC_V0.md`'s convention, for a shared vocabulary across PROPULSION/SYSTEMS documents): **MFR** (manufacturer-published spec), **SOURCED** (documented figure not from the core vendor), **EXTRAP** (extrapolated from a related/analog source), **ASSUM** (an assumption this document makes), **ROM** (first-principles rough-order-of-magnitude estimate, no hardware selected).

---

## 0. Status of DG-004 — what this document is, and is not, standing on

**SYSTEMS' own position:** conditional concurrence at **~700V DC nominal** was offered on AER-669 (2026-09-06), against PROPULSION's sourced case in `COMBO3_CORRECTED_ELECTRIC_MACHINE_V1.md` §2 (EMRAX 268 MV winding, BAMOCAR D3-700/400 controller, TE Connectivity Kilovac/Hartman switchgear class). Two conditions were attached: (1) the gate record must state plainly that ~700V is a deliberate step off the MIL-STD-704 standards map, not a free upgrade — 28V and ±270V are the only tiers with a published aerospace power-quality standard; (2) this concurrence exists specifically to unblock this document.

**What has NOT happened:** a **formal PROPULSION/Bill acknowledgment** that DG-004 is closed at ~700V. AER-671's own scope names this as an input to block on, not assume. As of this writing no comment from PROPULSION or Bill has landed on AER-669 or AER-609 responding to the concurrence. This document proceeds anyway, per AER-671's explicit instruction to publish a first-pass architecture "pending final DG-004 ratification" — that instruction is authorization to start work, not a substitute for the ack itself. **DG-004 is not closed. Treat every number below as provisional on that closure**, the same way `GENSET_PACKAGE_SPEC_V0.md` treats its own ~700–800V working assumption.

**A cross-cutting risk this document did not create but must name plainly:** `PROPULSION_RESET_V1.md` (AER-608, merged 2026-09-05/06) finds that **A2 — the series-hybrid architecture the genset-feed tap below is written for — does not close the mission with any core in PROPULSION's current catalog**, independent of DG-004, independent of battery chemistry, and independent of SAFETY's R1 ruling. Bill has an open `request_confirmation` on AER-608 (posted 2026-09-05T18:47, unanswered as of this document) choosing among A0 (direct-drive, no HV bus at all), A1 (parallel hybrid — boost motor + reserve battery, HV bus but no genset), and A2 (series hybrid — genset + twin motors, the architecture that uses every tap below). STRUCTURES' own review (AER-609/AER-670) adds that **no mount station has ever been sited for a genset core**, so A2 also carries an unsized structural gap on top of the power-closure gap.

**How this document handles that risk:** it is structured so the bus voltage, protection scheme, grounding, and LV tap (§§2–7) are **architecture-common** — A1 also runs a boost motor and reserve battery on this same HV bus, just without a genset feed. Only §3.1 (genset feed) is A2-specific and would be deleted, not reworked, if A2 is not funded. This is the cheapest way to avoid wasting the parts of this document that survive either outcome, but it does not change the fact that funding real mount/switchgear-placement design effort for a genset feed today is premature until AER-608 resolves — flagged again in §9.

---

## 1. Method

First-principles HV distribution architecture per Moir & Seabridge (*Aircraft Systems*), consistent with the SYSTEMS profile's standing method anchor. Because no qualified aerospace power-quality or insulation-coordination standard exists above the MIL-STD-704 ±270V ceiling (per the electrical-load tradecraft memory), §6's insulation/creepage/arc-fault margin borrows from the nearest applicable published bodies — IEC 60664-1 (insulation coordination for low-voltage systems) and SAE J1673 (electric/hybrid-vehicle propulsion wiring) — as analogs, not inherited certification. The protection scheme (§4) borrows precharge/HVIL/isolation-monitor practice from the same automotive/industrial HV-EV body of practice, for the same reason: it is the closest real, currently-fielded engineering discipline operating at this voltage class, even though it was not written for aircraft.

---

## 2. Bus voltage and operating envelope

| Parameter | Value | Basis | Tag |
|---|---|---|---|
| Nominal voltage | **~700V DC** | SYSTEMS' AER-669 conditional concurrence, against PROPULSION's `COMBO3_CORRECTED_ELECTRIC_MACHINE_V1.md` §2 sourcing | ASSUM (concurred, not yet formally ratified) |
| Upper protection ceiling | 780–800V | BAMOCAR D3-700/400 overvoltage cutoff (controller's own protection trip, not a design target) | MFR |
| Absolute test ceiling | 833V | EMRAX 268 MV winding absolute test voltage — never a normal operating point | MFR |
| Recommended normal regulation band | ~630–770V (±10% class, typical HV-EV bus regulation practice) | First-principles pick — keeps normal operation with margin below the 780V protection trip and above the low end needed for full-power delivery | ASSUM |
| Lower bound (battery SOC sag) | **Not sourced this pass — open item.** | Depends on the reserve battery's series cell count, which depends on the still-open chemistry choice (LFP, 208 kg, vs. ProLogium-class solid-state, 105 kg — both live in `PROPULSION_RESET_V1.md` §3) | **Blocked on PROPULSION**: need the pack's series-cell count once chemistry + capacity are fixed, to confirm the bus stays inside BAMOCAR's 12–700V input window across the full charge/discharge swing, not just at nominal. |

---

## 3. Topology

Single distribution bus, one busbar pair, with each consumer/source on its own contactor-protected tap — no tap wired through another tap's contactor. This is a direct application of the single-point-of-failure discipline SAFETY already forced onto the A1 reserve chain (`RESERVE_DEGRADED_ANNUNCIATION_V1.md`): a shared contactor path would mean one component's fault can silently take another's supply with it, exactly the failure class AER-610 P4 exists to catch.

**Taps, architecture-tagged:**

| Tap | Architecture | Contactor | Current (700V nominal) |
|---|---|---|---|
| Genset feed | **A2 only** | 1× main | ≈137 A (see §3.1) |
| Battery reserve tie | A1 and A2 | 1× main + 1× precharge | ≈57 A sustained (reserve draw); higher on regen/absorb transients, not sized this pass |
| Motor-controller tap(s) | A1: ×1 (boost motor); A2: ×2 (propulsion motors) | 1 per motor | A1: boost-delta only, not sized here (climb-boost power not yet published by PROPULSION for A1's actual boost split); A2: ≈111 A each (see §3.3) |
| LV DC-DC tap | A1 and A2 | 1× (fused, not a full contactor — see §3.4) | ≈4 A on the HV side at 2 kW/700V |

### 3.1 Genset feed (A2-contingent)

Feeds the genset's EMRAX 268 MV + BAMOCAR D3-700/400 output onto the bus. Steady-state target: **96 kW continuous** (mission cruise-bus floor, carried in from `GENSET_PACKAGE_SPEC_V0.md` §2, itself traced to AER-68/AERO).

- **Current at 700V nominal: 96,000 W / 700 V ≈ 137 A.** This is a refinement of `COMBO3_CORRECTED_ELECTRIC_MACHINE_V1.md`'s own 800V-based figure (≈120 A) — at the now-concurred 700V nominal rather than the wider 700–800V band that document used, current runs ≈14% higher. Still comfortably inside the BAMOCAR D3-700/400's 200 A_RMS continuous rating and the Kilovac/Hartman 500A-class contactor family. **This 14% correction should be carried back into any future revision of `COMBO3_CORRECTED_ELECTRIC_MACHINE_V1.md`'s current table**, flagged to PROPULSION in §10.
- Duty cycle: climb, cruise, and (per `GENSET_PACKAGE_SPEC_V0.md` Operational Context) descent as a battery-charging trickle. Not active Ground/Preflight, Engine Start (warm-up, no load acceptance per the ICD's state machine), or Post-flight.
- **This tap does not get built until A2 is funded.** Per §0, do not spend real switchgear-placement or harness-routing effort on it ahead of AER-608's resolution.

### 3.2 Battery reserve tie-in

- **Main contactor + precharge contactor/resistor pair.** Standard HV-EV practice, not currently named in any MAOS-ICE document: before the main battery contactor closes, a precharge resistor (through a smaller-rated precharge contactor) limits inrush current into the DC-link capacitance of up to three BAMOCAR D3-700/400 units (genset controller + up to two motor controllers) sitting on the bus. Without precharge, closing a 700V main contactor directly into discharged DC-link capacitors draws a current spike limited only by cable/contactor resistance — the standard failure mode this avoids is contactor tip welding on the very first energization. ASSUM (first-principles addition, not sourced from any MAOS document).
- **Reserve draw:** ~40 kW at 700V ≈ **57 A**, using `PROPULSION_RESET_V1.md` §0.3's current reserve baseline (26 kWh at 40 kW, ~39 min powered descent from FL180) — this supersedes the older "40 kWh/30 min" figure my own `LV_ELECTRICAL_LOAD_LIST_V1.md` §2 cites as the HV reserve doctrine precedent; flagged as a housekeeping correction in §9, not a change to any LV-side number (the LV list only needed the reserve battery's *existence* as a Source A precondition, not its capacity).
- Contactor/cable sizing for this tap should be set by the higher of sustained reserve current (57 A) and any charge-side current the genset trickle-charges the pack at during descent — not sized this pass, named open item.

### 3.3 Motor-controller taps

- **A2 (2× propulsion motors):** climb floor 155 kW combined ÷ 700V ≈ **221 A combined, ≈111 A per motor** (2× EMRAX 268 MV + BAMOCAR D3-700/400 each). Same 700V-vs-800V refinement as §3.1 applies (COMBO3's 800V figure: 146A/motor; this document's 700V figure: 111A/motor — **the correction runs the other direction here**, because COMBO3's per-machine figure was already computed at the lower 700V-equivalent current in its own worked table; re-verify against PROPULSION's original derivation before treating this as a discrepancy rather than confirmation. **Open item, named for the record rather than silently reconciled** — see §9.)
- **A1 (1× boost motor, EMRAX 228 + BAMOCAR D3-700/400):** climb-boost delta only, not full climb power (the ICE still supplies its own share, per the parallel-hybrid physics floor in `PROPULSION_RESET_V1.md` §1). **PROPULSION has not published the actual boost-power split for A1** — this tap's current is not sized in this document. Named as a consumed input SYSTEMS needs from PROPULSION before this tap can be closed for A1.

### 3.4 LV DC-DC tap

Confirms Source A from `LV_ELECTRICAL_LOAD_LIST_V1.md` §2 now that HV nominal voltage is fixed.

| Parameter | Value | Basis | Tag |
|---|---|---|---|
| Sizing point | Worst steady-state LV total across all flight phases: **54.5 A @ 28V ≈ 1,526 W** (Landing/Rollout, `LV_ELECTRICAL_LOAD_LIST_V1.md` §5) | Reuses the ELA matrix's own worst-phase total rather than re-deriving | ROM (inherits the LV list's own confidence tagging) |
| Recommended continuous rating | **~2 kW** (≈30% margin over the 1.53 kW steady peak, headroom for measurement/derating error, not for the named momentary transients below) | First-principles margin pick | ASSUM |
| **Not covered by this rating:** starter current (LV-17, 150–300A momentary if a dedicated LV starter is required) and gear retract/extend (LV-19, 0–30A momentary) | — | Both are named open items in the LV list itself (§7 items 1–2) and are too large/transient to fold into a continuous-duty DC-DC converter sizing. If LV-17 resolves toward a dedicated LV starter, that current needs its own path (a second, higher-current, momentary-duty tap or a direct battery-crank circuit), not this converter. | — |
| Input voltage range | Must span the full HV bus swing, including the still-open low end (§2) | — | **Blocked on §2's open item** |
| Output | 28V DC nominal | Matches LV bus definition, `LV_ELECTRICAL_LOAD_LIST_V1.md` §2 | — |
| Mass | **~2 kg ROM** | Typical automotive/industrial HV-EV accessory DC-DC converter power density at this class (~1 kW/kg, liquid- or forced-air-cooled COTS units) — no vendor selected | EXTRAP |
| Volume | **~1.8 L ROM** | Typical brick/module-format packaging for a 2kW-class unit at this voltage | EXTRAP |
| Efficiency (assumed) | ~92% | Typical class figure for a HV-to-LV isolated DC-DC converter | ASSUM |
| Waste heat | ~170 W at full continuous load | Derived from the efficiency assumption above | ROM |
| Cooling | Air-cooled class assumed adequate at this power level (see §7) | — | ASSUM |
| Location | **Not fixed — pending STRUCTURES envelope**, same open item as the rest of the HV switchgear group | — | **Blocked on STRUCTURES** |
| Vendor | **Not selected.** Same open-item class as the main contactor part number (AER-671's own scope names both as open). | — | Open |

---

## 4. Protection scheme

- **Main/motor/genset contactors:** TE Connectivity Kilovac/Hartman class, 500A continuous @ up to 900Vdc, AS9100-built, UL-recognized — concurring with PROPULSION's own sourcing in `COMBO3_CORRECTED_ELECTRIC_MACHINE_V1.md` §2. This document does not re-derive that family; it adopts it as the working candidate for every tap in §3. **Specific part number still open** (named in AER-671's own scope) — PROPULSION/SYSTEMS joint follow-up.
- **Precharge circuit** ahead of the battery main contactor (§3.2) — first-principles addition, not previously named in any MAOS-ICE or MAOS-ECS document.
- **HV Interlock Loop (HVIL):** a low-voltage loop routed through every HV connector's interlock pins in series; opening any HV connector (maintenance disconnect, a chafed/severed harness, a crash-structure separation) breaks the loop and commands every contactor open, without needing a smart controller to detect the fault. Standard automotive/industrial HV-EV practice (the SAE J1673 wiring-practice body this document already borrows from in §6). ASSUM/EXTRAP — no aerospace standard to cite, per the standards-gap framing in §1.
- **Isolation/ground-fault monitor (IMD):** a continuous insulation-resistance monitor across the HV bus, tripping a caution (not necessarily a shutdown — see §8's annunciation note) when insulation resistance falls below a threshold (industry-typical class figure ~100–500 Ω/V, ASSUM/EXTRAP, not sourced to a specific vendor this pass). This is the direct answer to AER-671's arc-fault-margin ask: an IMD catches an insulation-degradation precursor (chafed cable, contaminated connector, developing short) before it becomes an arc fault, which is the standard automotive/industrial HV-EV mitigation for exactly this risk class. Requires the bus to be a floating (ungrounded) DC system — see §5.
- **Fault/short-circuit protection:** fusing sized to each tap's continuous current (§3) with a defined interrupt rating; the Kilovac/Hartman contactors' own breaking capacity vs. available fault current at 700V needs verifying against the published datasheet once a specific part number is selected — **named open item, not verified this pass.**

---

## 5. Grounding/bonding scheme

Extends the independence principle already stated in `GENSET_PACKAGE_SPEC_V0.md` §2 ("engine block and generator housing bonded to airframe ground independently") to every HV bus participant:

- **Case/chassis bonding:** genset/generator housing, battery pack case, and each motor/controller housing bonded to airframe structure **independently** — not daisy-chained through another component's ground path, and not through the HV power-return conductor. This is the same single-point-of-failure discipline as §3's per-tap contactor rule, applied to grounding instead of power distribution.
- **HV bus itself is floating (ungrounded)** on the power conductors — standard automotive/industrial HV-EV practice, and the precondition that makes the IMD (§4) meaningful. Only case/chassis bonding is grounded to airframe; the DC+ and DC− power conductors are not referenced to airframe ground at any point. ASSUM — first-principles pick given the standards gap, consistent with the broader automotive HV-EV convention this document already borrows from.
- **Cable shielding:** HV cabling shielded per the existing EMRAX/BAMOCAR installation practice already cited in `COMBO3_CORRECTED_ELECTRIC_MACHINE_V1.md`; shield grounded at one end only, standard practice to avoid a shield-current ground loop. ASSUM.

---

## 6. Insulation/creepage/arc-fault margin — first-principles exercise

Named explicitly on AER-669 as the standards-gap cost of operating above the MIL-STD-704 ±270V "in-standard" ceiling — there is no qualified aerospace power-quality spec above 270V to inherit this from (see the electrical-load tradecraft memory). This section borrows from the two nearest applicable published bodies rather than inventing figures from nothing, and tags every borrowed number EXTRAP because neither source was written for an aircraft:

- **IEC 60664-1** (insulation coordination for equipment within low-voltage systems — creepage/clearance vs. working voltage and pollution degree) and **SAE J1673** (electric/hybrid-vehicle propulsion wiring) are the two bodies of practice this document leans on, matching the same automotive/industrial HV-EV analog this document already uses for §4's precharge/HVIL/IMD practice.
- **ROM creepage/clearance at 700–800V class, pollution degree 2** (enclosed installation, not directly exposed to conductive dust/moisture — a reasonable assumption for switchgear inside an airframe pod, not a verified environmental classification): creepage on the order of **8–12 mm** (reinforced insulation, material group IIIa/PCB-class), clearance on the order of **5.5–8 mm** through air. These are ROM/EXTRAP figures reflecting commonly-used industrial 800V-class EV practice, not a pulled IEC 60664-1 table — a real table lookup is needed once actual connector/busbar hardware is chosen, named as an open item below.
- **Working design margin:** recommend ≥1.5× the bare IEC minimum for any harness/busbar routing in the airframe, given a GA installation's vibration, humidity, and occasional-wetting environment is a harsher installation class than a sealed EV underbody pack — ASSUM, first-principles margin pick, not derived from a published table.
- **Arc-fault mitigation, first-principles best practice (no aerospace standard to cite):** physical separation of HV harness runs from LV/CAN runs (avoid parallel runs; cross at 90° where unavoidable), orange HV cable jacket convention (SAE J1673) for build/maintenance identification, and mechanical chafe protection (conduit or equivalent) at any point an HV harness crosses a structural bulkhead or a surface with relative motion.
- **What actually closes this section:** a dielectric-withstand (hi-pot) test and an insulation-resistance test on the real harness/connector set once parts are selected — commonly ~2×V_nom + 1000V for a hi-pot test in this practice class, ASSUM figure, not performed here. Named as a verification open item, same discipline as `GENSET_PACKAGE_SPEC_V0.md` §6's bench-rig-closes-the-analysis pattern.

---

## 7. Cooling/thermal interaction with the genset coolant loop

`GENSET_PACKAGE_SPEC_V0.md` §4 bounds the engine coolant loop at **≈90–110 kW rejection, ~105–110°C ceiling** (automotive/motorsport SI-engine coolant standard). This section asks whether HV switchgear/DC-DC hardware can share that loop or that thermal budget, per AER-671's own scope.

**Finding: almost certainly two separate coolant loops, not one.** The BAMOCAR D3-700/400 controllers (genset controller if A2, plus one or two motor controllers) are liquid-cooled per their own datasheet class (8.5 kg, 135 kVA continuous). Industrial servo-drive/IGBT-class power electronics of this class commonly spec a coolant inlet ceiling well below an SI engine's — typically on the order of **≤50–65°C**, ASSUM/EXTRAP, since BAMOCAR's own coolant-temperature spec was not pulled this pass (named open item below). Feeding a 105–110°C engine coolant loop into electronics rated for ≤65°C would trip the controller's own overtemperature protection or shorten IGBT/capacitor life; conversely, running a single loop cooled to electronics-safe temperatures would undersize the engine's own heat rejection, which is sized against the higher ceiling specifically because a higher-temperature loop rejects more heat per unit radiator area. **This is a genuine addition to `GENSET_PACKAGE_SPEC_V0.md` §4's cooling-package mass allowance (~30–40 kg), not covered by it as currently scoped** — that allowance is written for the engine coolant loop alone.

- **Contactors/busbars:** I²R losses at the currents in §3 (111–221A through 500A-class hardware) are modest relative to the switchgear's own rating — convection/air-cooling is very likely adequate without a dedicated coolant loop. ROM, not sized to a specific wattage this pass, not a claim on either coolant loop.
- **DC-DC converter:** ~170W dissipation (§3.4) — air-cooled class unit, no coolant-loop draw.
- **Consumed input needed, named:** BAMOCAR D3-700/400's actual coolant inlet-temperature and flow-rate spec (not sourced this pass — the manual reference in `COMBO3_CORRECTED_ELECTRIC_MACHINE_V1.md` covers electrical/mass specs, not its thermal/coolant interface). **Blocked on PROPULSION/vendor datasheet pull** before a second coolant loop can be sized rather than argued qualitatively.

---

## 8. Mass / power / volume / duty cycle rollup

Per the SYSTEMS profile's standing contract — every system mass/power figure carries its duty cycle and its assumptions, not a bare number.

| Item | Mass | Power (continuous) | Volume | Duty cycle | Tag |
|---|---|---|---|---|---|
| Main/genset/motor contactors + busbars/fusing/HVIL | *(carried in PROPULSION's existing BOM, `COMBO3_CORRECTED_ELECTRIC_MACHINE_V1.md` §3 row 6: ~15 kg, $7,000 — concurred with, not re-derived here)* | Contactor coil hold, ~1–3W each, negligible | Not separately volumed by either document — open item | Live whenever HV bus is energized (Engine Start → Shutdown) | EXTRAP (PROPULSION's figure) |
| Precharge resistor/contactor | ROM, small (<0.5 kg) — not separately counted in PROPULSION's line | Negligible (only draws during the ~1s precharge transient) | ROM, small | Momentary, at every HV bus energization | ROM |
| Isolation monitor (IMD) | ROM, ~0.3 kg | A few watts | ROM, ~0.1 L | Continuous whenever HV bus is energized | ROM |
| LV DC-DC converter (Source A) | ~2 kg | ~2.2 kW input / ~2 kW output at full load; ~170W dissipated as heat | ~1.8 L | Live whenever the genset is running (Engine Start → Shutdown), output tracks the LV load-vs-phase matrix in `LV_ELECTRICAL_LOAD_LIST_V1.md` §5 | EXTRAP |
| **Total SYSTEMS-owned incremental hardware, beyond PROPULSION's existing switchgear BOM line** | **~2.8 kg ROM** | **~2.2 kW peak input (DC-DC converter dominant)** | **~2 L ROM (excludes contactor group, not separately volumed by either document)** | — | Mixed, see rows above |

**What this table does not yet include, named rather than omitted silently:** the genset-feed tap's own contactor/harness mass (A2-contingent, folded into PROPULSION's existing 15kg line, not separately broken out); the second (electronics) coolant loop from §7 (mass entirely unsized pending PROPULSION's BAMOCAR coolant-spec pull); A1's boost-motor tap current/hardware (§3.3, blocked on PROPULSION's boost-split figure); and the reserve-tie contactor/cable sizing beyond the 57A sustained figure (§3.2, charge-side current not sized).

---

## 9. Open issues and risks

| # | Issue | Owner | Action |
|---|---|---|---|
| 1 | DG-004 not formally acknowledged by PROPULSION/Bill — this document runs on SYSTEMS' own AER-669 concurrence only | PROPULSION / Bill | Post explicit acknowledgment or counter-proposal on AER-669 or AER-609; this document is provisional until then |
| 2 | AER-608's architecture pick (A0/A1/A2) is unresolved — a live `request_confirmation` to Bill, unanswered as of this document. A2 does not close the mission on paper; only §3.1 of this document is A2-specific and would be deleted, not reworked, if A2 is not funded | Bill | Resolve AER-608's request_confirmation; do not commit real switchgear-placement or harness-routing effort to §3.1 until then |
| 3 | Reserve battery series-cell count / chemistry not fixed — blocks confirming the bus's low-voltage bound (§2) inside BAMOCAR's 12–700V input window | PROPULSION | Publish once chemistry (LFP vs. ProLogium-class) and capacity are fixed |
| 4 | STRUCTURES firewall/nacelle/switchgear envelope not sited — blocks HV harness routing and switchgear/DC-DC physical placement (§3.4, §4) | STRUCTURES | Named in AER-671's own scope as a consumed input; per AER-670, STRUCTURES has not sited a genset-core mount station at all, so this may itself be blocked on AER-608 |
| 5 | Switchgear vendor/part number not selected — TE Connectivity Kilovac/Hartman class is the working candidate, no specific part chosen | PROPULSION / SYSTEMS joint | Named in AER-671's own scope |
| 6 | A1's boost-motor tap current not sized — PROPULSION has not published the boost-power split (full climb power vs. ICE's own share) | PROPULSION | Needed to close §3.3 for A1 |
| 7 | BAMOCAR D3-700/400 coolant inlet-temperature/flow spec not sourced — blocks sizing the second (electronics) coolant loop named in §7 | PROPULSION (vendor datasheet pull) | Needed before the ~30–40 kg `GENSET_PACKAGE_SPEC_V0.md` §4 cooling allowance can be corrected to include electronics cooling |
| 8 | Genset-feed and motor-tap current figures in §3.1/§3.3 refine `COMBO3_CORRECTED_ELECTRIC_MACHINE_V1.md`'s 800V-based table to this document's concurred 700V nominal — one direction (genset feed) increases current ~14%, the other (motor taps) needs re-verification against PROPULSION's original derivation rather than being treated as a silent discrepancy | PROPULSION / SYSTEMS | Reconcile on the next revision of either document |
| 9 | IEC 60664-1 creepage/clearance figures in §6 are ROM/EXTRAP, not a real table pull — needs a real lookup once connector/busbar hardware is chosen | SYSTEMS | Follow-on once hardware is selected |
| 10 | Hi-pot/insulation-resistance qualification test (§6) not performed — this document's insulation margin is analysis only | SYSTEMS (test program, once hardware exists) | Named as the closing verification step, same pattern as the genset ICD's bench-rig |
| 11 | Precharge circuit, HVIL, and IMD (§4) are first-principles additions not previously named in any MAOS-ICE or MAOS-ECS document — worth SAFETY's eye on whether any of these three should be a formal FHA line rather than an engineering-discipline default | SAFETY | Consult requested |
| 12 | Housekeeping: `LV_ELECTRICAL_LOAD_LIST_V1.md` §2 cites the superseded "40 kWh/30 min" HV reserve doctrine; the current baseline is `PROPULSION_RESET_V1.md`'s 26 kWh/40 kW (~39 min). Does not change any LV-side number. | SYSTEMS | Correct on the LV list's next revision |

---

## 10. Interfaces published

- **To PROPULSION:** the 700V-nominal current refinement (§3.1, §3.3, item 8 above); the LV DC-DC tap sizing (§3.4) confirming Source A's specification; the two-coolant-loop finding (§7) as a real addition to the genset cooling-package mass allowance; the precharge/HVIL/IMD protection-scheme additions (§4) as new BOM lines beyond the existing switchgear placeholder.
- **To Structures:** the switchgear/DC-DC/IMD group's incremental mass (~2.8 kg ROM, §8) and its unresolved volume/placement pending the firewall/nacelle envelope (§9 item 4); the insulation/creepage margin's environmental assumption (§6, pollution degree 2, enclosed installation) as a packaging constraint.
- **To Safety:** the protection scheme (§4) and grounding scheme (§5) as failure-mode/redundancy input — this is a single, non-redundant bus topology at this pass (§3), named as an open question rather than asserted safe; the precharge/HVIL/IMD additions flagged for FHA review (§9 item 11).
- **To Elon and the board:** DG-004 remains open pending formal PROPULSION/Bill acknowledgment (§0, §9 item 1); this document's genset-feed tap is contingent on AER-608's still-open architecture decision, and the rest of the document is written to survive either A1 or A2 being chosen.

---

## 11. Approvals

- **Engineering owner:** SYSTEMS (draft, this document)
- **Joint owner:** PROPULSION (DG-004, genset feed, boost-motor split, coolant-loop coordination) — requested, not yet reviewed
- **Structural reviewer:** STRUCTURES (envelope, mass) — requested, not yet reviewed
- **Safety reviewer:** SAFETY (protection-scheme FHA, single-bus redundancy question) — requested, not yet reviewed
- **Date:** 2026-09-06 (V0 draft)

---

*Version 0 — 2026-09-06. First-pass HV bus architecture at DG-004's ~700V DC nominal per AER-671: bus voltage/envelope (§2), topology with architecture-tagged taps (§3), protection scheme including precharge/HVIL/IMD (§4), grounding/bonding extending the genset ICD's independence principle (§5), a first-principles insulation/creepage/arc-fault margin exercise against the MIL-STD-704 standards gap (§6), the genset-coolant-loop-vs-power-electronics-coolant-loop finding (§7), and the mass/power/volume/duty-cycle rollup per the SYSTEMS profile's standing contract (§8). DG-004 is not yet formally closed by PROPULSION/Bill, and the genset-feed tap is contingent on AER-608's still-open architecture decision — both named plainly rather than assumed (§0, §9).*
