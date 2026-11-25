# Calibrating a syringe pump by gravimetry

Delivered flow depends on two things: **how far the drive head travels** (the *pump mechanics* — the
carriage that pushes the syringe plunger) and the **syringe bore** you program. Weighing the water
measures only their *product*, so calibrate them **in order** — do it backwards and a mechanics error
hides inside a wrong "effective diameter":

1. **Pump mechanics first** (Step 1) — so the drive head moves the true distance.
2. **Syringe diameter second** (Steps 2–5) — the balance then corrects the bore alone.

The gravimetric part needs **only an analytical balance** — water weighs ~1 g/mL, so **1 mg ≈ 1 µL**:

```
Q_real = mass / (ρ_water · time)
D_real = D₀ · √(Q_real / Q_set)
```

If the balance reads **more** than programmed → the syringe is **wider** than you told the pump →
increase the diameter. **Two web calculators** (and `calibrate.py`) do the math — one for the
**mechanics** (Step 1) and one for the **diameter** (Steps 2–5); you just type the numbers you measured.

---

## Equipment

- **Analytical balance, 1 mg** (0.001 g); 0.1 mg is even better.
- **Dial indicator** or digital caliper (0.01 mm) — to measure drive-head travel (Step 1).
- Syringe under test, filled with **bubble-free** distilled water.
- **Parafilm** + a small receiving tube (e.g. Eppendorf).
- Tweezers/tongs for handling calibration weights (skin oil weighs enough to matter).
- The syringe pump.

## Step 0 — Calibrate the balance (once)

1. Place it on a stable, **level** table (centre the bubble with the feet); no drafts, sun or
   vibration. **Warm it up for ≥ 1 hour** powered on.
2. Empty pan, reading `0.0`. Enter external calibration (on CGOLDENWALL/JoanLab HZ-type
   balances: `Set` → menu → **`e-cal`**; it requests a standard weight, e.g. `cal-200` = 200 g).
3. Place the exact standard weight **with tongs**, wait, remove when prompted → returns to `0.0`.
4. **Verify**: place a known weight (e.g. 100 g) and check it reads correctly.

## Step 1 — Calibrate the pump mechanics (plunger travel)

> **⚙️ First, match the software to this pump.** `gearbox`, `rod` (lead screw) and `microstep` are
> stored **per pump** and must match the physical hardware you built — set them on the **keypad menu**
> or with `set_gearbox` / `set_rod` / `set_microstep`. **Re-check them whenever you swap pumps, or
> move the controller between a geared and an ungeared pump.** On this lab's pumps: ungeared →
> `gearbox 1:1`; geared → `gearbox 100:1`; both use the T8 **1-star (2 mm)** rod and `microstep 1/8`,
> with the same syringe diameter. A wrong `gearbox` shows up below as a clean ~25× / 100× travel error,
> a wrong `rod` as ~4× (see the troubleshooting table).

➡️ **[Open the mechanics calculator](software/calibration/calculator-mechanics.html)** — enter flow, time, diameter and start/end readings → expected travel, error, corrected THREADROD.

First check the mechanics on their own: *does the drive head move the distance it should?* This is the
**displacement method** of EURAMET CG-27 §4.3.3 (lab version: laser interferometer; workshop version:
a caliper or dial indicator — exactly how a 3D-printer axis's steps/mm is calibrated).

This pump has **no "move N mm" command** — you set a **diameter** and a **flow** and let it run, so the
travel comes from `flow × time`:

1. Mount a **caliper or dial indicator** against the drive head / carriage and **zero it**. Keep a
   syringe mounted, dispensing to waste, as in a normal run (metrology labs use a rigid glass/steel
   syringe with the mirror on the pusher block; a normal syringe is fine here).
2. Set the **diameter D** and a **flow Q** and run for a few minutes — **longer = smaller % error**
   (on the keypad, or `set_diameter`, `set_flow`, `set_state` in `pump_comm.py`). The commanded travel is
   ```
   L_set = Q · t / (π · (D/2)²)
   ```
   e.g. D = 9.05 mm, Q = 3000 µL/h, t = 15 min → **L_set ≈ 11.7 mm** (keep units consistent; the
   pump's max is 6000 µL/h, and the gearbox does **not** change this travel).
3. Read the **actual travel `L_real`**. Repeat 2–3× and average to shrink the reading scatter.
4. If `L_real ≈ L_set`, the mechanics are good. Otherwise correct the **lead-screw pitch** (`THREADROD`,
   via `set_threadrod`): `threadrod_corr = threadrod × (L_real / L_set)`. The firmware derives steps/mm
   from `microstep`, `gearbox` and `threadrod`, so `threadrod` is the knob to trim.

**No caliper?** Even a **ruler** works — just run longer so the move is ~50–100 mm and the coarser
reading stays a small fraction of the distance.

**If `L_real` comes out very different from `L_set`** — a *clean factor* points to a wrong setting; a
small % is just real tolerance or reading noise:

| `L_real` vs `L_set` | Likely cause | Fix |
|---|---|---|
| ~25× or ~100× off | **Gearbox** set wrong | set it to the pump's real ratio (1:1 / 25:1 / 100:1) |
| ~4× off | **Rod** wrong (1-star = 2 mm vs 4-star = 8 mm) | set rod to match the physical lead screw |
| ~2× / 4× / 8× off, gearbox & rod correct | **Microstep** mismatch (driver ≠ firmware) | check microstep — over UART it's normally self-consistent, so this is rare |
| small (< ~2 %) | lead-screw pitch tolerance or reading noise | tiny `THREADROD` tweak, or repeat 2–3× and average |
| erratic / not repeatable | loose coupling grub screw, binding, or missed steps | tighten the coupling, check for binding, lower speed / raise motor current |

The web calculator flags these automatically — if the error is large it tells you which parameter to
check.

## Step 2 — Prepare the gravimetric run

1. Fill the syringe, **purge all air** (a single bubble ruins a run). Mount it in the pump.
2. Program the **nominal diameter D₀** (caliper the inner bore, or use the syringe spec) and the
   target flow **Q_set**.
3. Let the pump drip into **any waste tube** until the flow is **steady** — **we use 5 min**; the
   first minutes clear residual bubbles and let the mechanics settle.

## Step 3 — Collect the output, weighed (timed collection)

**Prepare the collection tube first:** close the empty Eppendorf, seal the top with **parafilm**,
pierce a **small hole** with the **sharp tip of scissors** (just big enough for the capillary — the
thin outlet tube from the syringe — keeps
evaporation low), and stand it on a **small, very light base** (we cut one from packing foam —
negligible mass, keeps the round-bottom tube upright on the pan). **Weigh it in this final state**
(tube + parafilm + hole + foam base) → `m_empty`, or tare to zero.

**Then collect, without stopping the flow:** with the flow already steady, move the capillary from
the waste tube into the collection tube through the parafilm hole, and **start the stopwatch at that
instant**. Collect for an **exactly timed interval `t`**. Move the capillary out, press the parafilm
closed over the hole, and weigh immediately → `m_full`. Δm = m_full − m_empty.

<table>
<tr>
<td align="center" width="25%"><img width="200" alt="step 1" src="https://github.com/user-attachments/assets/76a64963-dd73-4e40-815d-11dcb9445f08" /><br><sub>1 — Caliper the bore to get the nominal diameter D₀</sub></td>
<td align="center" width="25%"><img width="200" alt="step 2" src="https://github.com/user-attachments/assets/36ad7822-de1d-44a0-88d6-b6ef6476418c" /><br><sub>2 — Seal the tube top with parafilm</sub></td>
<td align="center" width="25%"><img width="200" alt="step 3" src="https://github.com/user-attachments/assets/1dcda40d-8cfe-4803-887a-38633ac9baae" /><br><sub>3 — Pierce a small hole with a scissor tip</sub></td>
<td align="center" width="25%"><img width="200" alt="step 4" src="https://github.com/user-attachments/assets/a4ea4f2d-0760-4617-b317-4fd9e5fd26e1" /><br><sub>4 — Insert the capillary through the hole</sub></td>
</tr>
</table>

Aim for **Δm ≥ 100 mg** (e.g. 10 min at 1000 µL/h ≈ 167 mg → <1 % uncertainty at 1 mg).

## Step 4 — Compute the corrected diameter

➡️ **[Open the diameter calculator](software/calibration/calculator.html)** — Steps 2–5: type the weighed values, get the corrected diameter.

```bash
python3 calibrate.py --d0 8.94 --qset 1000 --mass 170.5 --minutes 10 --temp 20
#   flow real (balance): 1024.8 uL/h  (+2.5 %)
#   CORRECTED DIAMETER : 9.05 mm   <- enter this on the pump
```

Enter `D_real` on the pump. Repeat **one point** to confirm Q_real ≈ Q_set.

## Step 5 — Verify linearity

Repeat at **at least 3 flow points — the ones you'll actually use — with 3 measurements each** (e.g.
500 / 1000 / 2000 µL/h ×3), following EURAMET CG-27. The 3 repeats give the repeatability; if all
points return the same `D_real`, the calibration is solid and flow-independent.

---

## Background (optional)

Delivered flow = **drive-head speed × syringe cross-section**. Gravimetry sees only the product, so a
wrong lead-screw pitch looks exactly like a wrong bore — that's why Step 1 (travel) comes before
Step 4 (bore). Many open-hardware pumps skip the split and fold both into one "effective diameter"
per pump (Wijnen & Pearce, 2014): accurate flow, but you no longer know which part was off.

> **Mechanism cycle.** A lead-screw pump's flow has a slow ripple at the screw's turn period
> (EURAMET CG-27). Collect over **≥ 2 full turns** to average it out — it's expected, not a bubble.

## Practical tips (what ruins a run)

- **Evaporation**: at 1000 µL/h an open tube loses ~1 mg/min — always parafilm, and weigh
  immediately after capping.
- **Weights with tongs**, never fingers.
- **Exclude the first ~5 min** of each run (stabilization + bubble clearing).
- **Record water temperature**; the tool adjusts density (20 °C → 0.99821 g/mL). ±2 °C is
  negligible (<0.1 %).
- **Bubbles**: purge before, and if the output suddenly spikes/dips erratically, there is air in the
  line — stop and purge.

---

## Optional: cross-check with a flow sensor (higher precision)

The balance fixes the diameter. If you also have an **in-line flow sensor**, the same runs let you
check — and calibrate — the sensor against the balance (ground truth), then use the sensor as a
**continuous** flow reference without weighing every time. `calibrate.py` reports the sensor factor
if you pass `--sensor-ulh <reading>`.

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
*Tools: web calculator (`calculator.html`, mechanics + gravimetric) and `calibrate.py` · rev 2026-11-25*
