# Calibrating a syringe pump by gravimetry

A syringe pump computes plunger speed from the **syringe inner diameter** you program. Real
syringes — 3D-printed or commercial — have diameter tolerance, so the **delivered flow can be
off by several percent** even if the pump moves perfectly. In our tests a "3 mL" syringe with a
nominal 8.94 mm bore actually delivered **+2.5 %**; the corrected diameter was 9.05 mm.

> Delivered flow = *drive-head speed* × *syringe cross-section*, so there are **two** error sources:
> the **syringe diameter** (below) and the **pump mechanics** (how far the plunger really travels).
> Gravimetry alone can't tell them apart. On a trustworthy pump (e.g. Harvard PHD 2000) the diameter
> correction below is enough; on a **home-built pump** calibrate the mechanics too — see
> [Two errors in one](#two-errors-in-one-syringe-diameter-vs-pump-mechanics).

This gravimetric calibration needs **only an analytical balance** — no flow sensor. Water
weighs ~1 g/mL, so **1 mg ≈ 1 µL**: the balance gives real dispensed volume directly.

```
Q_real = mass / (ρ_water · time)
D_real = D₀ · √(Q_real / Q_set)
```

If the balance reads **more** flow than programmed → the syringe is **wider** than you told
the pump → increase the diameter. Enter `D_real` and the pump delivers true flow.

The calculator `calibrate.py` does the math — you just type the numbers you measured.

---

## Equipment

- **Analytical balance, 1 mg** (0.001 g); 0.1 mg is even better.
- Syringe under test, filled with **bubble-free** distilled water.
- **Parafilm** + a small receiving tube (e.g. Eppendorf).
- Tweezers/tongs for handling calibration weights (skin oil weighs enough to matter).
- The syringe pump (this project).
- *For the pump-mechanics check (optional, see below):* a **dial indicator** or digital
  caliper (0.01 mm) to measure drive-head travel.

## Step 0 — Calibrate the balance (once)

1. Place it on a stable, **level** table (centre the bubble with the feet); no drafts, sun or
   vibration. **Warm it up for ≥ 1 hour** powered on.
2. Empty pan, reading `0.0`. Enter external calibration (on CGOLDENWALL/JoanLab HZ-type
   balances: `Set` → menu → **`e-cal`**; it requests a standard weight, e.g. `cal-200` = 200 g).
3. Place the exact standard weight **with tongs**, wait, remove when prompted → returns to `0.0`.
4. **Verify**: place a known weight (e.g. 100 g) and check it reads correctly.

## Step 1 — Prepare the run

1. Fill the syringe, **purge all air** (a single bubble ruins a run). Mount it in the pump.
2. Program the **nominal diameter D₀** (caliper the inner bore, or use the syringe spec) and
   the target flow **Q_set**.
3. Let the pump drip into **any waste tube** until the flow is **steady** — **we use 5 min**;
   the first minutes clear residual bubbles and let the mechanics settle.

## Step 2 — Collect the output, weighed (timed collection)

**Prepare the collection tube first:** close the empty Eppendorf, seal the top with
**parafilm**, pierce a **small hole** with the **sharp tip of scissors** (just big enough for the
capillary — keeps evaporation low), and stand it on a **small, very light base** (we cut one from
the foam used to pack boxes — negligible mass, keeps the round-bottom tube upright on the pan).
**Weigh it in this final state** (tube + parafilm + hole + foam base) → `m_empty`, or tare to zero.

**Then collect, without stopping the flow:** with the flow already steady (Step 1), move the
capillary from the waste tube into the collection tube through the parafilm hole, and **start the
stopwatch at that instant**. Collect for an **exactly timed interval `t`** (`t` goes straight into
the result). Move the capillary out, press the parafilm closed over the hole, and weigh immediately
→ `m_full`. Δm = m_full − m_empty.

<table>
<tr>
<td align="center" width="25%"><img width="200" alt="step 1" src="https://github.com/user-attachments/assets/76a64963-dd73-4e40-815d-11dcb9445f08" /><br><sub>1 — Caliper the bore to get the nominal diameter D₀</sub></td>
<td align="center" width="25%"><img width="200" alt="step 2" src="https://github.com/user-attachments/assets/36ad7822-de1d-44a0-88d6-b6ef6476418c" /><br><sub>2 — Seal the tube top with parafilm</sub></td>
<td align="center" width="25%"><img width="200" alt="step 3" src="https://github.com/user-attachments/assets/1dcda40d-8cfe-4803-887a-38633ac9baae" /><br><sub>3 — Pierce a small hole with a scissor tip</sub></td>
<td align="center" width="25%"><img width="200" alt="step 4" src="https://github.com/user-attachments/assets/a4ea4f2d-0760-4617-b317-4fd9e5fd26e1" /><br><sub>4 — Insert the capillary through the hole</sub></td>
</tr>
</table>

Aim for **Δm ≥ 100 mg** (e.g. 10 min at 1000 µL/h ≈ 167 mg → <1 % uncertainty at 1 mg).

## Step 3 — Compute and correct

➡️ **[Open the web calculator](software/calibration/calculator.html)** — type the weighed values, get the corrected diameter (no install).

```bash
python3 calibrate.py --d0 8.94 --qset 1000 --mass 170.5 --minutes 10 --temp 20
#   flow real (balance): 1024.8 uL/h  (+2.5 %)
#   CORRECTED DIAMETER : 9.05 mm   <- enter this on the pump
```

Enter `D_real` on the pump. Repeat **one point** to confirm Q_real ≈ Q_set.

## Step 4 — Verify linearity

Repeat at **2–3 flows** (e.g. 500 / 1000 / 2000 µL/h) with the corrected diameter. If they all
give the same `D_real`, the calibration is solid and flow-independent.

---

## Two errors in one: syringe diameter vs. pump mechanics

Gravimetry measures the **delivered volume** — the end-to-end truth. But that number lumps two
physically independent error sources together (EURAMET CG-27; IEC 60601-2-24):

```
delivered flow  =  drive-head speed   ×   syringe cross-section area
                   └─ pump mechanics ─┘    └── syringe geometry ──┘
```

- **Syringe geometry** (the diameter): bore tolerance is **1 % (large syringes) to 3 % (small)**.
  Scales flow by a *constant factor*, independent of flow rate and of position along the barrel.
- **Pump mechanics** (how far the plunger actually travels per command): lead-screw pitch,
  steps/mm, microstep linearity, coupling backlash/compliance. Commercial drivers spec **±1–2 %**.

The diameter-only calibration above (Steps 1–4) folds **both** into an *effective diameter*. That
is fine when the pump mechanics are trustworthy — e.g. a **Harvard PHD 2000 (±0.35 %)**. For a
**home-built pump** (stepper + lead screw) you cannot assume that: a steps/mm error disguises
itself as a diameter error, and gravimetry alone **cannot tell them apart** (both scale flow by the
same constant factor).

### How the pump is calibrated in the literature

Most open-hardware pump papers **do not measure the plunger travel directly** — they compute the
step size from geometry and finish with an **end-to-end gravimetric calibration that folds mechanics
*and* geometry into a single constant**:

- **Geometric starting point** — steps (or microsteps) per unit volume follow from the drivetrain:
  ```
  µsteps/µL = (steps_per_rev × microstepping) / (π · (ID/2)² · lead)
  ```
  e.g. a Marlin pump with 200 steps/rev, 16 µsteps, an 8 mm/rev screw and a BD 10 mL syringe
  (ID 14.5 mm) → **2.42 µstep/µL**, set in firmware with `M92 E2.42` (Baden et al., 2024).
- **Gravimetric finish** — Wijnen, Hunt, Anzalone & Pearce (*Open-Source Syringe Pump Library*,
  PLoS ONE 2014): set the calibration to a nominal value, **dispense a volume twice, weigh the
  second push, divide actual / commanded, enter the ratio, repeat 3×**. One number now captures
  the whole chain.

The catch — and the whole point of this section — is that **this does not separate the two errors**.
A lead-screw whose real pitch differs from nominal is indistinguishable from a wrong bore, so it is
silently absorbed into an *effective, per-pump inner diameter* (several 3D-printed-pump studies
report using a device-specific ID for each pump for exactly this reason). That is fine if you only
want accurate flow. It is **not** fine if you want to know *which part* is off — for that you must
measure the travel on its own.

### Separating the two — the "displacement method"

EURAMET CG-27 §4.3.3 names this the **displacement method**: mount a mirror on the pusher block and
measure its **travelled distance directly**, independent of the syringe — `Q = (x₂−x₁)/Δt · πr²` —
which captures the *"unloaded movement of the syringe driver head"* (Batista et al., 2020 [1]). The
lab-grade version uses a **laser interferometer** (nanometre resolution); the workshop version is the
same measurand with a **dial indicator / caliper** (~0.01 mm) — the familiar **3D-printer axis
steps/mm calibration** applied to the pump carriage (Method A). If you have a reference pump instead
of a dial indicator, Method B gets you there without disassembly.

> **Caveat (from the same guide).** The displacement method measures the *unloaded* travel. With a
> real syringe, **rubber-stopper compression and drivetrain compliance** add a further error the
> travel alone cannot see. So the full picture is **travel (mechanics) + gravimetry (loaded,
> real syringe)**: the gap between them *is* the stopper/compliance contribution.

#### Method A — Measure plunger travel directly (no syringe)

This checks the mechanics on their own: *does the drive head move the distance you command?*

1. Mount a **dial indicator** (or digital caliper) against the drive head / carriage and **zero it**.
   Remove the syringe or leave it empty — this test is purely mechanical.
2. Command a **long move** of known length `L_set` (e.g. **20.00 mm**). Long travel makes the
   reading error (±0.01–0.02 mm) a negligible fraction.
3. Read the **actual travel `L_real`** off the indicator.
4. Mechanical error = `L_real / L_set − 1`. Correct it (inverse):
   - Firmware with a **steps/mm** constant → `steps_per_mm_corr = steps_per_mm × (L_real / L_set)`.
   - Repeat over the **full stroke** (0→20, 20→40 mm …) to catch lead-screw non-uniformity.
5. *(Optional) Backlash:* command a move, then **reverse**; the dead distance before the indicator
   starts moving is the backlash. Only matters if you ever reverse — infusion-only runs don't.

After this the pump delivers the commanded **distance**. Any gravimetric error that remains is then
genuinely the **syringe diameter**, which you fix with Steps 1–4.

#### Method B — Cross-check against a reference pump (same syringe)

If you have a trustworthy reference pump (Harvard PHD 2000) you can split the error without a dial
indicator, because the **syringe geometry is common to both pumps and cancels**:

1. Run the **same syringe** on the **reference pump**, gravimetric → error `ε_ref` ≈ syringe
   geometry (reference mechanics are ±0.35 %).
2. Run the **same syringe** on the **pump under test**, same `Q_set`, gravimetric → error `ε_total`.
3. **Pump-mechanics error ≈ `ε_total − ε_ref`.**

Method A is absolute and needs only a caliper; Method B needs a reference pump but no disassembly.
Use **A to calibrate a new pump build**, **B to audit a pump already in service**.

### Order of operations

1. **First calibrate the mechanics** (Method A) → the pump moves the true distance.
2. **Then calibrate the diameter** (Steps 1–4) → the flow is true.

In this order the diameter you end up with is the *real* bore, not a number that silently absorbs a
steps/mm error.

> **Mechanism cycle.** On a lead-screw pump the delivered flow carries a slow periodic ripple at the
> screw's revolution period ("mechanism cycle time", EURAMET CG-27). Time each gravimetric
> collection over **≥ 2 full revolutions** of the screw to average it out. This ripple is expected —
> it is not a leak or a bubble.

---

## Practical tips (what ruins a run)

- **Evaporation**: at 1000 µL/h an open tube loses ~1 mg/min — always parafilm, and weigh
  immediately after capping.
- **Weights with tongs**, never fingers.
- **Exclude the first ~5 min** of each run (stabilization + bubble clearing).
- **Record water temperature**; the tool adjusts density (20 °C → 0.99821 g/mL). ±2 °C is
  negligible (<0.1 %).
- **Bubbles**: purge before, and if the output suddenly spikes/dips erratically, there is air
  in the line — stop and purge.

---

## Optional: cross-check with a flow sensor (higher precision)

The balance fixes the diameter. If you also have an **in-line flow sensor**, the same runs
let you check — and calibrate — the sensor against the balance (ground truth), and then use
the sensor as a **continuous** flow reference without weighing every time. `calibrate.py`
reports the sensor factor if you pass `--sensor-ulh <reading>`.

For the flow-sensor hardware, firmware and high-rate acquisition, see the Rio controller:
➡️ **[wenzel-lab/rio-controller](https://github.com/wenzel-lab/rio-controller)** (pressure/flow module).

---

## References

**Metrology standards & guides**

- **EURAMET**, *Calibration Guide No. 27 — Guidelines for the Calibration of Drug Delivery Devices
  and Infusion Device Analysers*, v2.0 (02/2026). E. Batista (IPQ), M. Benkova (CMI), H. Bissig
  (METAS), O. Büker (RISE). The master guide: gravimetric method (§4.3.1), **displacement method
  (§4.3.3)**, front-tracking (§4.3.2), micro-PIV (§4.3.4), mechanism cycle time, response time.
  <https://www.euramet.org/Media/docs/Publications/calguides/I-CAL-GUI-027_Calibration_Guide_No._27_web.pdf>
- **IEC 60601-2-24**, *Particular requirements for the basic safety and essential performance of
  infusion pumps and controllers* — trumpet curve (2/5/11/19/31-min windows), start-up graph,
  short-term flow accuracy.
- **AAMI TIR101:2021**, *Fluid delivery performance testing for infusion pumps.*
- **ISO 7886-2**, syringe cross-sectional-area tolerances (≈1 % large, 3 % small syringes).

**Primary methods (peer-reviewed)**

1. **E. Batista et al.** (2020), *"Development of an experimental setup for microflow measurement
   using interferometry"*, **Flow Measurement and Instrumentation 75, 101789** — the displacement
   method behind §4.3.3 (mirror on the pusher block; 5000 mL/h → 1.6 nL/min, U = 0.9–2 %).
   <https://www.sciencedirect.com/science/article/pii/S0955598620301345>
2. **H. Bissig et al.** (2015), *"Primary standards for measuring flow rates from 100 nL/min to
   1 mL/min — gravimetric principle"*, **Biomed. Tech. 60(4), 301–316** (METAS) — the primary
   gravimetric standard the guide builds on.
3. **E. Batista, R. F. Martins, V. Silvério, I. Godinho** (2025), *"Advancing calibration techniques
   for accurate micro and nanoflow measurements"*, **Frontiers in Nanotechnology** — recent review
   comparing gravimetric, interferometric, front-track and pending-drop methods with uncertainties.
   <https://doi.org/10.3389/fnano.2025.1600426>

**Open-hardware pump calibration**

4. **B. Wijnen, E. J. Hunt, G. C. Anzalone, J. M. Pearce** (2014), *"Open-Source Syringe Pump
   Library"*, **PLoS ONE 9(9):e107216** — end-to-end "push twice, take the ratio" volumetric
   calibration. <https://doi.org/10.1371/journal.pone.0107216>
5. **Baden et al.** (2024), *"3D Printer-Controlled Syringe Pumps…"*, **arXiv:2402.04354** —
   `µsteps/µL` from drivetrain geometry; Marlin `M92 E` + gravimetric finish.
   <https://arxiv.org/abs/2402.04354>

> **About EURAMET / IPQ.** EURAMET is the European Association of National Metrology Institutes — the
> body that defines SI-traceable measurement in Europe. IPQ (*Instituto Português da Qualidade*) is
> Portugal's national metrology institute; **Elsa Batista (IPQ)** leads both the guide and the
> displacement-method papers above.

---
*Tool: `calibrate.py` · rev 2026-11-24*
