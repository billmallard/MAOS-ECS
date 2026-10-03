# MAOS Ice-Protection Method v0 — Fluid vs. Electrothermal vs. Waste-Heat Regime Curves

**Status:** Draft v0 — method anchor + parameterized regime curves. **Not a recommendation.** Two inputs are still owed before this closes into an absolute mass/power number (§6).
**Subsystem:** SYSTEMS — ice protection (method selection is a live, undecided SYSTEMS-owned trade per the SYSTEMS profile)
**Date:** 2026-10-03
**Related issue:** [AER-2571](/AER/issues/AER-2571) — self-initiated per the SYSTEMS execution contract. [AER-673](/AER/issues/AER-673) (PROPULSION: ack DG-004 + review `HV_BUS_ARCHITECTURE_V0.md`) and [AER-674](/AER/issues/AER-674) (SAFETY: FHA of the HV protection scheme) have sat in backlog, unpicked, since 2026-09-06; PROPULSION/SAFETY/AERO/STRUCTURES have been status-only since 2026-09-18. This document is the slice of the ice-protection trade that needs neither hand-off to close — the method anchor and the per-unit-area/per-unit-span regime curves, parameterized so the two still-missing numbers (AERO's protected area, PROPULSION's waste-heat-available figure) are drop-in multipliers, not hardcoded assumptions.
**Data provenance tags** (`HV_BUS_ARCHITECTURE_V0.md`'s convention, for a shared vocabulary): **MFR** (manufacturer-published spec), **SOURCED** (documented figure, not core-vendor), **EXTRAP** (extrapolated from a related/analog source), **ASSUM** (an assumption this document makes), **ROM** (first-principles rough-order-of-magnitude estimate, no hardware selected).

---

## 0. Why this exists, and what it is not

MAOS is Experimental Amateur-Built. Nothing in this document is a certification claim — 14 CFR 25 Appendix C is cited below as the best publicly available, physically grounded characterization of icing severity, borrowed as a **design input** the way the HV bus work already borrows MIL-STD-704 voltage tiers and IEC 60664-1 insulation practice as analogs rather than inherited compliance (`HV_BUS_ARCHITECTURE_V0.md` §1, §6).

**This document is the AERO/PROPULSION-independent slice only.** Per AER-2571's scope, it does four things and stops there:

1. States the icing duty-cycle input as a design choice (§2), not a certification claim.
2. Lays out the Messinger-class mass-and-energy-balance method (§3) as the sizing anchor, parameterized so protected area and heat-available are drop-in multipliers applied *after* this document, not baked into it.
3. Produces per-unit-area / per-unit-span regime curves for fluid, electrothermal (cyclic de-ice vs. continuous anti-ice), and the bleed/hot-air-equivalent candidate (§4), each tied to the bus-voltage/generation-capacity floor it needs.
4. Names the two inputs still owed — AERO's protected-area figure, PROPULSION's waste-heat-available-vs-duty-cycle-overlap figure — as the explicit remaining step to an absolute number and a recommendation (§6). **Neither is guessed here.**

Per the SYSTEMS profile's standing method anchor, every system model here gets a sanity-check worked numerical example against a duty cycle and an energy balance before being trusted — §3.4 and §4 carry that check inline, flagged as illustrative ROM anchors, not final sizing.

---

## 1. Method anchor: Messinger-class mass-and-energy balance

First-principles ice-accretion/anti-ice sizing per the Messinger control-volume model (Messinger, 1953, *"Equilibrium Temperature of an Unheated Icing Surface as a Function of Air Speed"*, and the AIAA/NASA icing-literature family built on it) — a coupled mass balance (incoming supercooled water flux → collection efficiency → impingement rate) and energy balance (convective loss, evaporative loss, sensible heating of impinging droplets, kinetic/aerodynamic heating recovery, heater input or latent heat of fusion) closed on a control volume at the leading edge. This is the same method-anchor class as any duty-cycle-and-energy-balance sizing elsewhere in SYSTEMS' work (the BTMS design-point method, the LV load-vs-phase matrix) — applied here at the collection-efficiency/freezing-fraction resolution specific to icing. Moir & Seabridge (*Aircraft Systems*) and Gudmundsson (*General Aviation Aircraft Design*) are the surrounding-system/installed-penalty references, consistent with the SYSTEMS profile's standing bookshelf.

**Fidelity boundary, named up front:** this is a 1-D strip (per-unit-span, per-unit-area) model with uniform collection efficiency across the strip and no runback-into-unheated-zone modeling. It is the correct first-pass fidelity for a method/regime-curve document — full icing-CFD (multi-shot, 3-D collection efficiency) is the fidelity escalation once actual leading-edge geometry makes the uniform-strip assumption break down, the same discipline SYSTEMS already applies before reaching for OpenFOAM on a thermal problem. `maos-openfoam` carries an icing-capable solver path if that escalation becomes necessary once AERO's geometry lands; it is not needed for this pass.

---

## 2. Icing duty-cycle input — a design choice, not a certification claim

14 CFR 25 Appendix C defines the design icing atmosphere as two named envelopes, each a family of liquid water content (LWC) vs. mean effective droplet diameter (MVD) vs. temperature curves:

- **Continuous maximum (stratiform)** — Appendix C Figure 1 (LWC/MVD/temperature relationship) at a standard 17.4 nautical mile horizontal cloud extent, with Figure 3 giving the LWC correction factor for other extents.
- **Intermittent maximum (cumuliform)** — Appendix C Figure 4, standard 2.6 nautical mile extent, with Figure 6's correction factor for other extents.

Both represent the FAA's probable-maximum (99th-percentile) characterization of random icing encounters of "standard" extent — a certification basis for Part 25/29 aircraft, and, independent of that status, the best publicly available quantification of "how severe, for how long, at what droplet size" available to any icing-sizing method. **SOURCED** (structure of the envelope — two named conditions, their standard extents, and the Figure numbers — confirmed against the regulation text and FAA DOT/FAA/AR-00/30, which digitizes Appendix C into tabular/distance-based form).

**This document's design-point choice** (the actual LWC/MVD/temperature/altitude values used below) is pulled from secondary engineering literature characterizing the shape of Figures 1 and 4 (the values commonly reproduced in icing-engineering texts and FAA guidance), **not a digitized pull of the primary Figure 1/3/4/6 curves this pass**. Tagged **ASSUM** throughout, with the primary-source digitization named as an open item (§8) before these numbers back a final hardware commitment — the same discipline `HV_BUS_ARCHITECTURE_V0.md` §6 applies to its own IEC 60664-1 creepage figures ("ROM/EXTRAP... not a pulled table").

| Condition | LWC | MVD | T_amb | Altitude band | Exposure | Sizing role | Tag |
|---|---|---|---|---|---|---|---|
| Continuous max (stratiform) | 0.5 g/m³ | 20 µm | −10°C | ≤10,000 ft (GA unpressurized operating band, v1.0) | Sustained (duration = flight-phase length, not Appendix C's own 17.4 nmi cap) | Anti-ice sizing — the system must hold indefinitely, not just for one cloud pass | ASSUM, representative mid-envelope point |
| Intermittent max (cumuliform) | 2.0 g/m³ | 20 µm | −8°C | Same band | Short burst (2.6 nmi / TAS ≈ a few minutes) | De-ice sizing — bounds the worst-case accretion rate a cyclic system must shed per cycle | ASSUM, representative peak-envelope point |

Mission TAS is taken as **150 kt (≈77.2 m/s)**, a placeholder ROM for MAOS cruise-class performance pending AERO/Elon's actual performance figures — flagged, not a load-bearing number for this document's regime curves (TAS is a drop-in parameter in §3 like area is, it is simply instantiated here to produce the worked numerical examples in §4).

**Why this matters beyond the number itself:** both conditions sit well inside the regime where kinetic/aerodynamic heating is negligible (§3.3) — this is a slow-airplane icing problem, not a transonic one, and that fact alone rules out "let the airframe's own speed do some of the anti-ice work" as a real option here.

---

## 3. Parameterized governing equations

Symbols marked **[OWED]** are the two inputs this document deliberately does not hardcode — they multiply onto the per-unit-area/per-unit-span curves in §4 once supplied. Symbols marked **[THIS DOC]** are fixed by §2's duty-cycle choice.

| Symbol | Meaning | Units | Source |
|---|---|---|---|
| `LWC`, `MVD`, `T_amb`, `V`, altitude | Icing duty-cycle condition | kg/m³, µm, °C, m/s, ft | **[THIS DOC]** §2 |
| `A` | Protected surface area | m² | **[OWED — AERO]** |
| `s` | Protected span (leading edge length) | m | **[OWED — AERO]** |
| `r_LE` | Local leading-edge radius (per protected zone — wing, tail, prop, inlet) | m | **[OWED — AERO]**, sharper than "area" alone: both `h_c` and `β` below need it per zone, not just a lumped area figure |
| `β` | Local collection efficiency | dimensionless, 0–1 | Function of `MVD`, `r_LE`, `V` (Langmuir-distribution-class droplet-trajectory result). ROM range 0.5–0.8 for GA-scale leading edges (Gudmundsson/Moir & Seabridge class), used as a placeholder pending `r_LE` |
| `ṁ_imp''` | Impinging water mass flux | kg/s/m² | `= LWC · V · β` |
| `h_c` | Convective heat-transfer coefficient | W/m²K | Stagnation-point correlation, function of `V`, `r_LE`, local air density/viscosity (ISA at altitude) |
| `n₀` | Freezing fraction of impinging water | dimensionless, 0–1 | Solved from the energy balance below |
| `Q_available(t)` | Waste-heat available for a thermal tap, vs. time in the flight profile | W, vs. flight phase | **[OWED — PROPULSION]**, per §6 |

**Mass balance:** `ṁ_imp'' = LWC · V · β` — the water catch rate per unit area. Multiplying by `A` gives the total impingement rate once AERO's area lands; nothing here requires that multiplication to produce the per-unit-area curves in §4.

**Energy balance, continuous anti-ice** (surface held at `T_s ≥ 0°C`, "running wet" — the standard mode, not fully evaporative, per the note in §4.2):

```
q_heater'' = q_conv'' + q_evap'' + q_sens'' − q_kin''

q_conv''  = h_c · (T_s − T_amb)
q_sens''  = ṁ_imp'' · c_p,water · (T_s − T_amb)
q_kin''   = h_c · r · V² / (2 · c_p,air)        (r ≈ 0.9, recovery factor)
q_evap''  ≈ 0 for running-wet (water sheds as liquid, not evaporated); large if forced fully evaporative — see §4.2
```

**Energy balance, unheated accretion → freezing fraction** (cyclic de-ice sizing input — how much ice forms, and how fast, with no heater running):

```
n₀ = [q_conv'' − q_kin'' + q_sens''] / (ṁ_imp'' · L_f)        (evaluated at T_s = 0°C, the ice/water interface)
```

where `L_f = 334,000 J/kg` (latent heat of fusion of water). `n₀` is the fraction of the impinging water that freezes rather than running back as liquid film; `n₀ · ṁ_imp''` is the ice accretion rate per unit area, which sets how long a cyclic de-ice system can go between pulses before a given ice thickness has built up.

**None of the above three equations contain `A` or `s`.** They are intensity relationships — W/m² and kg/s/m² — by construction, per AER-2571's scope item 2. §4 instantiates them at §2's duty-cycle point to produce actual regime-curve numbers; multiplying by AERO's eventual area figure is the only step left to reach an absolute power/mass number.

---

## 4. Regime curves

All worked numbers below use §2's design point (continuous: LWC 0.5 g/m³, MVD 20 µm, −10°C; intermittent: LWC 2.0 g/m³, −8°C; both at 150 kt / ≈8,000–10,000 ft), `β = 0.65` (ROM mid-value), `r_LE = 0.015 m` (ROM GA-scale leading edge, **owed from AERO** per zone), giving `h_c ≈ 300 W/m²K` (Fröessling-type stagnation correlation, ROM). These are **illustrative anchor points for the method**, not a sizing commitment — re-run §3's equations the moment `r_LE`, `β`, and the Appendix C primary-table pull (§8) are tightened.

### 4.1 Fluid (TKS-class)

Dry hardware (pump, distribution lines, porous panels, tank structure) scales roughly with protected **span**, not duration; fluid mass scales with span × duration × flow rate. This is the method behind the "dry-hardware mass is power-cheap and duration-limited" framing already in the ice-protection tradecraft memory, now given actual per-span numbers:

| Parameter | Value | Basis | Tag |
|---|---|---|---|
| Normal-flow application rate | ≈8.4 kg/hr per meter of protected span (≈1.5 oz/min/ft) | Published CAV Systems/Cox & Company TKS porous-panel class application rate | EXTRAP (secondary-sourced; no panel vendor selected) |
| Max/high-flow rate (de-ice shedding mode) | ≈16.7 kg/hr/m span (≈2× normal) | Same source, high-flow setting | EXTRAP |
| Dry hardware mass | ROM, small and largely span-independent at MAOS scale (pump + lines + panel backing) — whole-airplane precedent ≈16–18 kg dry hardware class (TKS-class GA installations) | Prior ice-protection tradecraft survey (Plane+Pilot, Cox & Company, Aviation Consumer) | EXTRAP |
| Electrical draw | ROM ≈40 W (pump only) — power-cheap regardless of bus voltage/architecture | Same survey | EXTRAP |
| **Fluid mass vs. duration (the actual regime curve)** | `m_fluid = flow_rate · s · t` — e.g., at normal flow, a 5 m span over 30 min ≈ 21 kg; over 60 min ≈ 42 kg | Direct application of the rate above | ROM |

**Bus-voltage floor:** negligible — ≈40 W pump load clears any bus (28V LV, ±270V, or ~700V) with no architectural dependency. This is fluid's structural advantage over electrothermal: it does not touch DG-004 at all.

### 4.2 Electrothermal — continuous anti-ice

Worked at §2's continuous design point, `T_s = 0°C` (minimum anti-ice target):

| Term | Value | Note |
|---|---|---|
| `q_conv''` | 300 W/m²K × 10°C ΔT ≈ **3,000 W/m²** (≈279 W/ft²) | Dominant term |
| `q_sens''` | 0.0251 kg/s/m² × 4,186 J/kgK × 10°C ≈ **1,050 W/m²** (≈98 W/ft²) | Warming impinging droplets to surface temp |
| `q_kin''` | 300 × 0.9 × (77.2² / (2×1,005)) ≈ **800 W/m²** (≈74 W/ft²) | Recovery credit — small; GA speeds don't reach the regime where aero heating matters |
| **`q_heater'', running wet** | **≈3,250 W/m² (≈302 W/ft²)** | **This is the regime-curve number** — a pure intensity, independent of `A` |
| `q_heater'', fully evaporative | ROM **≈56,700 W/m²** (≈5,270 W/ft²), order of magnitude only | Evaporating the *entire* impingement catch costs the full latent heat of vaporization (2.26 MJ/kg) per unit water, not just sensible heat — this is the standard reason running-wet anti-ice, not evaporative, is the normal mode (Moir & Seabridge); **do not design to the evaporative number** |

All figures **ROM**, carrying `r_LE = 0.015 m` and `β = 0.65` placeholders — tagged for re-run once AERO's actual leading-edge radius lands.

### 4.3 Electrothermal — cyclic de-ice

Worked at §2's intermittent design point: `ṁ_imp'' = 0.002 × 77.2 × 0.65 ≈ 0.100 kg/s/m²`, `n₀ ≈ 0.15` (15% of impinging water freezes at this condition, per §3's freezing-fraction equation), giving an **ice accretion rate ≈ 0.015 kg/s/m² ≈ 1 mm/min** (at ≈900 kg/m³ ice density) — a plausible severe-icing accretion rate, consistent with Appendix C's intermittent-max envelope being a short, severe-burst condition by design.

- A cyclic system sized to shed before a thin (≈1–3 mm) bond layer forms at each heated zone implies a **cycle period on the order of 1–3 minutes** at this accretion rate — consistent with the standard SAE/Moir & Seabridge electrothermal de-ice cadence (parting strip continuously heated; de-ice zones cycled sequentially around the span).
- **Average power regime curve:** cyclic de-ice trades a high *peak* zone intensity (comparable to or higher than §4.2's running-wet anti-ice number, to melt the bond layer fast) against a **duty fraction of ≈10–25%** (only one zone heated at a time, briefly) — giving an **average draw of roughly 0.1–0.25× the continuous anti-ice intensity**, i.e. **≈325–810 W/m² (≈30–75 W/ft²) average**, not the full 3,250 W/m² continuous figure.
- **Simplification named:** this is the standard duty-fraction approximation used at this fidelity (not a full bond-layer-melt-energy model per zone) — adequate for a regime-curve comparison, not for sizing an actual heater-zone sequencing controller.

### 4.4 Bus-voltage / generation-capacity floor (DG-004 tie-in)

This is where the ice-protection method ties directly to the open DG-004 gate (`HV_BUS_ARCHITECTURE_V0.md`) rather than floating independently of it:

| Total continuous anti-ice load (illustrative — `A` not yet known) | Current @ ±270V (MIL-STD-704 in-standard ceiling) | Current @ ~700V (DG-004 candidate) |
|---|---|---|
| 5 kW (≈1.5 m² at §4.2's intensity) | 18.5 A | 7.1 A |
| 15 kW (≈4.6 m²) | 55.6 A | 21.4 A |
| 30 kW (≈9.2 m²) | 111 A | 42.9 A |

**The gate is generation capacity, not wire gauge.** A typical GA wing leading edge alone is commonly 3–6 m² before tail and prop de-ice are added — at §4.2's ≈3,250 W/m², that is already a 10–20 kW continuous draw for the wing alone, before any margin. A conventional 28V/±270V GA electrical architecture (alternator-limited, typically a few kW to low tens of kW total generation) cannot absorb that **and** run everything else — this is the finding already in the ice-protection tradecraft memory, now anchored to this document's actual regime-curve numbers rather than a qualitative claim. **DG-004's ~700V/≈96 kW-continuous genset-fed bus (`HV_BUS_ARCHITECTURE_V0.md` §3.1) is what reopens continuous electrothermal anti-ice as a real candidate on this airframe** — not because 700V makes the heater draw smaller in an absolute sense (power is power), but because a conventional LV/MEA-class GA generation budget was never going to carry a double-digit-kW continuous load on top of everything else, and the hybrid's genset-scale generation is the one architectural fact on this airframe that changes that. The secondary benefit — lower current, lighter harness, per the wire-sizing causal chain (load → protection device → conductor ampacity) — is real but not the gating factor.

**This section does not resolve DG-004.** It states what ice protection needs *from* that gate: if DG-004 closes below genset-class continuous generation capacity, or if A0/A1 (no genset, per `HV_BUS_ARCHITECTURE_V0.md` §0's open AER-608 architecture question) is chosen over A2, continuous electrothermal anti-ice should be re-scored down relative to fluid or cyclic de-ice, which do not depend on genset-class generation at all.

### 4.5 Bleed/hot-air equivalent and waste-heat-fed thermal

**True compressor-bleed anti-ice (the Part 25 turbine convention) does not apply to any of this airframe's candidate architectures (A0 direct-drive, A1 parallel hybrid, A2 series hybrid) — none has a gas-turbine core.** This is a structural fact about the airframe, not an open question: bleed-air wing anti-ice is a turbine-engine pattern, and piston/hybrid GA architectures that use any thermal anti-ice historically do it via an exhaust-shroud or similar waste-heat heat exchanger instead (cabin heat is the common piston-GA precedent; wing anti-ice via the same principle is rarer but not unprecedented).

The real candidate in this family, for this airframe, is **waste-heat-fed thermal anti-ice** — tapping genset exhaust, generator/motor losses, or the battery BTMS loop (already an established pattern on hybrid/turbine-electric architectures per the ice-protection tradecraft memory) to feed the same `q_heater''` requirement computed in §4.2, at near-zero *additional* power cost, in exchange for a heat-exchanger/plumbing/routing penalty. The method for using it once the owed number lands (§6):

```
Required: Q_available(t) at the protected zone's temperature class ≥ q_heater'' · A,
          for every t where the icing duty cycle (§2) and the flight-profile duty cycle overlap.
```

Two failure modes to check once PROPULSION's figure arrives, already named in the ice-protection tradecraft memory and repeated here because they are the actual gate on this option: (a) waste heat peaks during high-power climb/cruise, but icing exposure is not phase-locked to that — a duty-cycle mismatch kills the concept even if the *average* heat-available number looks sufficient; (b) the heat source may be too far from the protected leading edge for the ducting mass/pressure-drop penalty to still beat running an electric heater locally off the HV bus.

---

## 5. Regime comparison summary

| Candidate | Power (continuous or avg) | Mass driver | Bus-voltage dependency | Duration limit |
|---|---|---|---|---|
| Fluid (TKS-class) | ROM 40 W (pump only) | Fluid mass, linear in span × duration × flow rate (§4.1) | None — clears any bus | Finite — tank size trades directly against duration |
| Electrothermal, continuous anti-ice | ROM ≈3,250 W/m² (running wet) | Heater/wiring mass, scales with `A` | **Needs genset-class generation capacity** (§4.4) — effectively gated on DG-004 closing near A2's ~700V/~96 kW bus | Unlimited, while bus power holds |
| Electrothermal, cyclic de-ice | ROM ≈325–810 W/m² average (10–25% duty) | Heater/wiring mass, scales with `A`; lower average power than anti-ice | Eases the §4.4 gate by 4–10× vs. continuous, but peak zone intensity still needs provisioning | Unlimited, while bus power holds |
| Waste-heat-fed thermal | Near-zero *additional* electrical power, at a plumbing/ducting mass and routing-volume cost | Heat exchanger + ducting mass, not yet sized | Independent of DG-004's electrical gate — but gated on PROPULSION's waste-heat figure instead (§6) | Unlimited, subject to duty-cycle overlap (§4.5) |
| Bleed/hot-air (Part 25 convention) | N/A | N/A | N/A | **Does not apply — no turbine core in any candidate architecture (§4.5)** |

No ranking is offered here. Every row needs `A` (AERO) to become an absolute number, and the waste-heat row additionally needs PROPULSION's figure to be scored at all — see §6.

---

## 6. Inputs still owed — explicitly flagged, not guessed

| # | Input | Owner | Needed for | Status |
|---|---|---|---|---|
| 1 | Protected external surface area `A`, by zone (wing LE, tail LE, prop, inlet), **and local leading-edge radius `r_LE` per zone** (sharper than area alone — §3 needs it for `h_c` and `β`) | AERO | Converting every §4 per-unit-area regime curve into an absolute mass/power number | **Not received.** Do not substitute a guessed area — every absolute number in this document's §4.4 illustrative table is explicitly tagged "illustrative," not sized to a real area |
| 2 | Waste-heat available, as a rate **and temperature**, vs. time in the flight profile (not an average figure — per the thermal-management tradecraft lesson, a "waste heat available" number and a "BTMS rejection duty" are the same number from two ends; ask for whichever PROPULSION already computes) | PROPULSION | Scoring §4.5 (waste-heat-fed thermal) against electrothermal/fluid at all | **Not received.** `AER-673` (PROPULSION ack of DG-004 + HV bus review) has sat in backlog since 2026-09-06 and is the most likely vehicle to carry this alongside it |

**This document does not pick a method.** It cannot — picking requires both rows above, and item 1 alone gates every candidate's absolute number, not just one. What it does do is remove every other excuse for the trade to stay undefined: the method is anchored, the duty cycle is stated, and the regime curves are real numbers waiting on two specific multipliers.

---

## 7. Fidelity boundary and escalation path

This is a 1-D strip, uniform-collection-efficiency, steady-state model — adequate for a method/regime-curve pass and for comparing candidates, not for certifying a final heater-zone layout or a runback-icing analysis into unheated aft zones. Escalate to full icing-CFD (`maos-openfoam`'s icing-capable solver path, multi-shot, 3-D collection efficiency) only once AERO's actual leading-edge geometry (§6 item 1) makes the uniform-strip assumption the limiting error, not as a default next step. This mirrors the same fidelity discipline already applied to BTMS and cooling-drag problems in the SYSTEMS tradecraft memory.

---

## 8. Open issues and risks

| # | Issue | Owner | Action |
|---|---|---|---|
| 1 | §2's duty-cycle design point (LWC/MVD/temperature/altitude) is pulled from secondary icing-engineering literature characterizing Appendix C's shape, not a digitized pull of the primary Figure 1/3/4/6 curves | SYSTEMS | Digitize the actual Appendix C figures (or pull FAA DOT/FAA/AR-00/30's distance-based conversion) once a specific operating altitude band is fixed by the mission profile — needed before these numbers back a final hardware commitment, not before a method/regime-curve comparison |
| 2 | `r_LE` (leading-edge radius per protected zone) and `β` (collection efficiency) are ROM placeholders (0.015 m, 0.65) — both drive `h_c` and the entire §4 regime-curve table | AERO | Named in §6 item 1 as a sharper version of the "surface area" ask |
| 3 | Mission TAS (150 kt) is a placeholder for the worked numerical examples, not load-bearing on the method itself | AERO / Elon | Confirm actual cruise-class TAS once performance figures are published; re-run §4's worked numbers, not the governing equations in §3 |
| 4 | §4.4's bus-voltage floor table assumes DG-004 closes at ~700V on a genset-class (~96 kW continuous) bus — if AER-608's A0/A1/A2 architecture question resolves away from A2, the generation-capacity argument for continuous electrothermal anti-ice weakens accordingly | PROPULSION / Bill | Tracked on `HV_BUS_ARCHITECTURE_V0.md` §9 item 2; this document inherits that same open item rather than re-litigating it |
| 5 | Cyclic de-ice's average-power figure (§4.3) uses a duty-fraction approximation, not a full bond-layer-melt-energy-per-zone model | SYSTEMS | Adequate for regime comparison; revisit once a candidate is actually selected and a heater-zone sequencing design is needed |
| 6 | Waste-heat-fed thermal anti-ice (§4.5) cannot be scored against the other candidates at all without PROPULSION's figure | PROPULSION | Named in §6 item 2 |

---

## 9. Interfaces published

- **To AERO:** the sharpened area request (§6 item 1) — protected zone area **and** local leading-edge radius per zone, not area alone; the regime curves in §4 that are ready to multiply by that figure the moment it lands.
- **To PROPULSION:** the waste-heat-available-vs-duty-cycle-overlap request (§6 item 2), framed as "your own BTMS rejection-duty number, not a separate measurement" per the thermal-management tradecraft pattern; the §4.4 bus-voltage/generation-capacity finding that ties continuous electrothermal anti-ice's viability to DG-004's A2/genset-class outcome.
- **To Safety:** none yet — this document produces no mass/power commitment to review. Flagged for `AER-674`'s eventual scope once a method is actually selected and sized.
- **To Elon and the board:** the ice-protection method is anchored and the regime curves are real, but **the method selection stays open** per the SYSTEMS profile's standing instruction not to let the rest of the board assume an answer — fluid, continuous electrothermal, cyclic electrothermal, and waste-heat-fed thermal each remain live candidates with very different mass/power signatures (§5), and closing the choice needs §6's two inputs, not a guess.

---

## 10. Approvals

- **Engineering owner:** SYSTEMS (draft, this document)
- **Consuming reviewers:** AERO (area/geometry input owed, §6), PROPULSION (waste-heat input owed, §6; DG-004 generation-capacity tie-in, §4.4) — requested, not yet reviewed
- **Date:** 2026-10-03 (V0 draft)

---

*Version 0 — 2026-10-03. Self-initiated per AER-2571 to build the AERO/PROPULSION-independent slice of the ice-protection method while both hand-offs remain unpicked. Anchors the Messinger-class mass-and-energy-balance method (§1, §3), states the Appendix C icing duty-cycle as a cited design choice rather than a certification claim (§2), and produces per-unit-area/per-unit-span regime curves for fluid, continuous and cyclic electrothermal, and waste-heat-fed thermal (§4), each tied to the bus-voltage/generation-capacity floor it needs (§4.4) and compared side by side (§5). Explicitly does not recommend a method — AERO's protected-area/geometry figure and PROPULSION's waste-heat-available figure are named as the two remaining inputs (§6), not guessed.*
