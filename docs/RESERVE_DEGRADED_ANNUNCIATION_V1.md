# A1 Powered-Descent Reserve — "RESERVE DEGRADED" Annunciation (AER-610 P4)

**Status:** Draft v1 — signal-exposure confirmation, OR/latch-logic ruling, panel design
**Subsystem:** SYSTEMS (bus-health signal chain, annunciation logic, panel placement) — joint with PROPULSION (raw-signal naming, SOC threshold)
**Date:** 2026-09-05
**Related issue:** AER-612 (child of AER-608; closure condition P4 of AER-610)
**Scope:** A1 (in-line parallel hybrid) only. Answers, for `PROPULSION_RESET_V1.md` §3.1 P4: can the four raw fault signals actually be gotten off the chosen hardware; where does the OR-logic live; what does the panel/EFIS annunciation look like.

## 0. The ask, restated

SAFETY (AER-610) ruled that A1's four-component powered-descent reserve chain — reserve battery, HV bus/contactor, boost-motor windings, motor controller — can fail silently (the ICE keeps flying the airplane with no thrust loss and no cue), and required a single latching **"RESERVE DEGRADED"** caution, OR-triggered by any of the four raw faults. PROPULSION owns which four raw signals must exist and the SOC threshold; SYSTEMS owns confirming the hardware can expose them, ruling on where the OR-logic lives, and the annunciation design.

## 1. Signal-exposure confirmation

A1's reserve chain, per `PROPULSION_RESET_V1.md` §3, is: 26 kWh LFP-or-ProLogium reserve pack → BMS + contactor → EMRAX 228-class boost motor + UNITEK BAMOCAR-PG-D3-700/400 controller. Both boxes are real, currently-documented COTS hardware with CAN interfaces. Checked against their published manuals this pass:

| # | Raw signal | Source hardware | Mechanism | Evidence |
|---|---|---|---|---|
| 1 | Pack SOC below reserve threshold | BMS (Orion BMS 2 or Elithion Lithiumate class — both already catalogued in `POWERTRAIN_TRADE_STUDY_50K_V1.md` §9 as candidates) | BMS calculates SOC by coulomb counting + open-circuit-voltage drift correction and publishes it on up to 15 programmable CAN messages (dual CAN 2.0B) — "virtually all BMS parameters" are CAN-exposable per vendor manual | **MFR** — Orion BMS 2 Operation Manual (Ewert Energy Systems, rev 1.0, 2018-01-09), "State of Charge Calculation" and "CANBUS Communication" §§ |
| 2 | HV bus/contactor continuity fault (reserve path) | Same BMS | The BMS drives the reserve contactor via a **Contactor Enable Output** and independently *verifies* the command was obeyed: after commanding an output off, it monitors for continued current flow; if current hasn't stopped within ~500 ms it sets a **relay enforcement fault DTC (P0A06/P0A07)** and latches all outputs off. This is a current-flow-based obey-check, not a literal auxiliary-contact weld-check — functionally it proves the contactor executed the command, which is what "continuity" needs for this purpose, but the distinction is worth keeping straight (see §5, open item) | **MFR** — same manual, "Digital On/Off Outputs," "Contactor Enable Output," "Digital on/off Relay failsafe" §§ |
| 3 | Boost-motor winding fault | EMRAX 228 motor + BAMOCAR controller | EMRAX motors carry an embedded KTY 81-210 stator-winding thermistor (max winding temp 120°C per EMRAX manual v5.4). The BAMOCAR controller reads it and raises fault code **7 (MOTORTEMP)** on its own fault register, exposed over its CAN-BUS interface (ISO 11898) as well as its front-panel LED + 7-segment display | **MFR** — UNITEK BAMOCAR-D3 manual ("Fault Indication," "CAN-BUS" §§); EMRAX Motor/Generator manual v5.4 (2020-03-05), winding-sensor spec |
| 4 | Motor-controller fault | BAMOCAR controller | The same fault register carries controller-side codes independent of the motor itself: POWERFAULT (2), BUS TIMEOUT (4), POWERVOLTAGE (5), DEVICETEMP (8), I_PEAK (9), RACEAWAY (A), HW_FAIL (D), and others — any of these is a genuine "motor controller fault flag," CAN-exposed the same way as MOTORTEMP | **MFR** — same BAMOCAR manual |

**Conclusion: yes.** All four raw signals PROPULSION named are things the currently-catalogued reserve-chain hardware can actually expose over CAN, using documented, named fault mechanisms already built into COTS boxes this program has already priced — not a capability that has to be invented or specified into a future part.

## 2. Ruling: where the OR-logic lives

**The OR (and latch) logic lives in avionics — the FIX-Gateway compute layer — not in any single component's onboard firmware.** This is close to a forced conclusion given the actual hardware, not a preference:

- **No single node sees all four signals.** The BMS sees SOC and its own contactor-enforcement fault; the BAMOCAR controller sees MOTORTEMP and its own controller faults. Neither box natively receives the other's CAN traffic.
- **Both boxes are closed COTS firmware.** BAMOCAR-PG-D3 and Orion BMS 2 / Elithion Lithiumate are commercial servo-drive and BMS products; SYSTEMS cannot add cross-source aggregation logic inside either one's firmware. "Put the OR-gate in the bus controller" is not an available design choice with the hardware this program has actually selected.
- **FIX-Gateway is the one layer that already has all four.** It is the open-source (GPLv2) "central avionics data broker" (`github.com/billmallard/fix-gateway`) that already ingests CAN-FIX from every source on the bus into one named-parameter database, and it ships a documented plugin architecture (`fixgw/plugins/compute.py`) built for exactly this class of derived value — it already computes pressure/density altitude, wind triangle, cross-track error, and several N-input aggregate functions (`average`, `sum`, `max`, `min`, `span`, `select`) from raw published parameters the same way this caution would need to be computed.
- **The gap is real and already flagged in that code.** `compute.py` has no boolean-OR or latching aggregate function today, and its own source carries a standing `# TODO: Add a check for Warns and alarms and annunciate appropriatly` (line 1094) — this issue is the concrete instance of that TODO, not a new ask invented here.

This matches the issue's own framing ("a bus-health/avionics logic item, not propulsion hardware") — the hardware facts just make it unambiguous rather than a matter of taste.

## 3. Design: the RESERVE DEGRADED signal chain

### 3.1 FIX-Gateway: two new compute-plugin functions

`fixgw/plugins/compute.py`'s existing aggregate functions (`sumFunction`, `maxFunction`, etc.) all share one shape: take N named input keys, combine, write one output key, propagate the worst-case quality flags. Two more of the same shape close this gap:

- **`or`** — boolean OR across N discrete inputs (SOC-below-threshold, contactor-enforcement-fault, MOTORTEMP-fault, controller-fault — each already a 0/1 discrete once published as a CAN-FIX parameter). Structurally identical to the existing `maxFunction`, just without needing the numeric max itself.
- **`latch`** — one "set" input (the OR output above) and one "reset" input (a maintenance/pilot-ground action, not an in-flight control); once set goes true the output stays true regardless of what the inputs do afterward, until reset is asserted.

These compose as `latch(or(soc_low, contactor_fault, motor_temp_fault, controller_fault), ground_reset)` → one output key, e.g. `RESERVE_DEGRADED`.

**Why latch, not self-clear:** a transient contactor-enforcement fault or a momentary CAN dropout still means the reserve chain's state at that moment was unproven — if it recovers on its own, that does not retroactively prove it would have delivered power the instant the ICE actually failed. The caution's entire purpose is to catch a *silent* degradation the pilot has no other way to see; a self-clearing indication would silently re-hide the same failure mode it exists to catch. Reset belongs on the ground (a maintenance/preflight action), not to a cockpit control the pilot could press away mid-flight.

**CAN-FIX parameter home:** the canfix-spec parameter table already has an allocated-but-undefined slot, **ID 570–571 "Hybrid System Status" (WORD, discrete bits)**, in the same High-Priority Engine/Hybrid block as Main Battery Charge, Main Propulsion Bus Voltage, and Main Battery Current. The four raw fault bits and/or the final latched RESERVE_DEGRADED bit are a natural fit there rather than a newly-reserved ID — flagged to canfix-spec maintainers as a candidate use of that slot (§5).

### 3.2 pyEfis: panel widget

pyEfis already ships a single-state annunciator-light widget, `pyefis.instruments.pa.Panel_Annunciator` (`src/pyefis/instruments/pa/__init__.py`) — four states via `setState()`: 0 black/off, 1 yellow, 2 red, 3 green, plus a text label via `setWARNING_Name()`. **It is not currently wired into the screen builder** — unlike `data_annunciation`, there is no `build_panel_annunciator` factory function or registry entry in `screenbuilder_factory.py`, so today it cannot be placed on a screen from YAML config. Closing this needs one factory function mirroring the existing `build_data_annunciation` pattern (`status_path`/binding options → `pa.Panel_Annunciator(**kwargs)`), registered the same way, then bound to `RESERVE_DEGRADED` via the existing `LiveBind` boolean-semantics mechanism (`pyefis.instruments.live_binding`) that already drives other bindable widget settings from a FIX key.

**State mapping:** Mode 1 (yellow) when `RESERVE_DEGRADED` is true, Mode 0 (off) otherwise — yellow, not red, because this is a caution ("be aware, not immediate action required" — the ICE is still flying the airplane unassisted) rather than a warning, consistent with the SAE ARP4102/GA CAS caution/warning convention this program already treats as engineering best practice rather than a certification requirement on an E-AB airplane. Because the underlying key is latched (§3.1), the widget will not need its own latch behavior — it faithfully displays whatever FIX-Gateway holds, and FIX-Gateway holds it until ground reset.

**Panel placement:** in the primary flight-instrument field of view, not an engine/systems subpage a pilot has to navigate to. The entire point of this caution is a cue a pilot catches without looking for it, specifically so that if the ICE later fails, the pilot already knows whether the reserve is there. The current default PFD layout (`src/pyefis/config/screens/pfd.yaml`) has no existing master-caution/CAS annunciator area at all — its only annunciation today is a small, deliberately subtle navdata-currency flag overlaid at the top-left corner of the attitude indicator (row 2, column 2). This means placing RESERVE DEGRADED establishes the pattern rather than fighting one: recommend a dedicated, non-subtle annunciator position near the top of the primary display (adjacent to or above the airspeed/altitude tapes), sized and colored to be seen in a glance, not read for detail — the opposite treatment from the navdata flag, which is designed to be ignorable until tapped.

## 4. Answer to "what closes this"

1. **Signal exposure — confirmed.** All four raw signals are real, MFR-documented, CAN-exposed fault mechanisms on hardware already in this program's catalog (§1). Nothing needs to be invented or specified into a future part.
2. **OR-logic location — ruled.** Avionics (FIX-Gateway's compute plugin), not the bus controller or motor controller, because neither COTS box sees all four signals and neither is firmware SYSTEMS can extend (§2).
3. **Panel/EFIS design — specified.** Reuse the existing (currently unregistered) `Panel_Annunciator` widget, wire it through the screen builder, bind it to a latched `RESERVE_DEGRADED` FIX key, yellow caution state, placed prominently on the PFD rather than an engine subpage (§3).

Recommend `PROPULSION_RESET_V1.md` §3.1 P4 be updated to reflect this as **ANSWERED at the design-spec level**; implementation (the two `compute.py` functions, the `screenbuilder_factory.py` registration, the CAN-FIX parameter allocation) is tracked as open work below.

## 5. Open items (named, non-blocking to this ruling)

- **Exact CAN bit-level mapping.** This pass confirms both BAMOCAR and the BMS candidates *have* CAN-exposed fault registers with real MFR-documented fault codes, but the literal CAN ID/byte layout for each code lives in vendor documents not yet pulled (BAMOCAR's separate "-CAN Manual," referenced but not itself retrieved this pass; the Orion BMS 2 Software Utility manual's "Editing CAN Messages" section). PROPULSION or SYSTEMS follow-up to pull these before wiring the actual CAN-FIX bridge — does not change the ruling in §2 or the design in §3.
- **canfix-spec parameter allocation.** §3.1's proposed reuse of ID 570–571 ("Hybrid System Status") needs canfix-spec maintainer concurrence — a spec-governance step, not an engineering unknown.
- **Code implementation lives outside SYSTEMS' repo-write boundary.** `fix-gateway` and `pyEfis` are not among the repositories SYSTEMS can push to. This document is the design spec; the two `compute.py` functions and the `screenbuilder_factory.py`/`live_binding` wiring need to be implemented by whoever holds write access to those repos (flagging to Bill / PROPULSION for routing).
- **Contactor-continuity semantics.** Orion's relay-enforcement fault is a *current-flow-after-command* check, not a literal auxiliary-contact/weld-detection signal. It answers the same question ("did the contactor obey?") but if PROPULSION's eventual BMS selection lacks this specific failsafe behavior, re-verify against that vendor's manual rather than assuming it carries over.
- **Elithion Lithiumate as the BMS candidate** (the alternative to Orion in `POWERTRAIN_TRADE_STUDY_50K_V1.md` §9) is documented with "built-in contactor drivers with precharge" but its comms-loss/watchdog and enforcement-fault behavior was flagged there as needing a vendor RFQ/spec call — this signal-exposure confirmation currently rests on Orion BMS 2's published manual; re-confirm if Elithion is the one actually selected.
