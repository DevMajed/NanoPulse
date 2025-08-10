# NanoPulse – A Precision 24-bit Pulse Capture Digitizer
Hardware project — firmware will live in another repo.

# Purpose & Use Cases
**NanoPulse** is a 2-channel, 24-bit, trigger-synchronized digitizer for **µs-scale pulses**. It gives a cleaner, more deterministic “scope view” with higher vertical resolution, lower drift, and easy automation in a tiny form factor.
- What: 24-bit, 2-channel digitizer for µs pulses (hundreds of kHz BW).  
- For: labs validating pulse shape, battery DCIR, and rail transients, Ultrasonic/sonar echo logger (lab/test-tank, 40–300 kHz)
- Why: cleaner data than 8–12-bit scopes, deterministic trigger, tiny + scriptable.

<img width="1976" height="1343" alt="image" src="https://github.com/user-attachments/assets/27db298e-53ce-4721-a3fc-f0237d6d417a" />

---

# Purpose & Use Cases

**NanoPulse** delivers a deterministic external-trigger capture path tuned for **hundreds of kHz BW** and **µs pulses**.

## 1) Scope-replacement for pulsed benches (µs pulses)
**What it solves**  
Verify the **actual** applied pulse (rise/fall, overshoot, flatness, droop) without wheeling in a DSO.

**Who it’s for**  
Power-electronics / microwave labs, bench integrators.

**Hook**  
24-bit depth + clean CNV trigger reveals subtle flat-top errors and overshoot that 8–12-bit scopes smear.

**Connect (minimal extras)**
- `Sense Out (BNC/SMA)` → `NanoPulse IN` *(use a simple divider if needed)*
- `External Trigger (DDG/TTL)` → `TRIG IN` *(optional on-board 50 Ω)*

**Quick checklist**
- [ ] Minimal overshoot/ringing  
- [ ] Flat-top ripple/droop within spec  
- [ ] Repeated pulses overlay tightly (deterministic timing)

**Output**  
CSV + plots via simple Python scripts.

---

## 2) Battery / Cell DCIR & Pulsed-Load Logging
**What it solves**  
Clean capture of **voltage droop and relaxation** during programmed load pulses (≈10 µs–100 ms) for DCIR and transient analysis.

**Who it’s for**  
Battery research labs, EV startups, QA.

**Hook**  
High dynamic range + low drift → clearer droop and recovery curves than a typical scope; easy CSV export.

**Connect (minimal extras)**
- `Cell V` → `NanoPulse IN` *(add a 2-resistor divider if V exceeds input range)*  
- Optional: `Shunt V (current)` → `CH2`

**Quick checklist**
- [ ] Stable ΔV (droop) across repeats  
- [ ] Consistent relaxation time constants (τ)  
- [ ] Negligible baseline drift on long captures

**Output**  
CSV of V(t) (and I(t) if used), with auto-computed DCIR and τ.

---

## 3) Power-Rail / Load-Step Transient Verification (PMIC/LDO)
**What it solves**  
Measures **droop, settling time, and small ripple** on precision rails during load steps (≈100 µs–10 ms).

**Who it’s for**  
Analog/PMIC teams, embedded HW labs.

**Hook**  
24-bit vertical resolution + drift stability → see micro-ripple/settling that scopes miss.

**Connect (minimal extras)**
- `Rail` → `NanoPulse IN` *(use a small divider if needed)*  
- `Load-Step Trigger` → `TRIG IN` *(or use Manual Trigger)*

**Quick checklist**
- [ ] Droop within budget  
- [ ] No excessive undershoot/overshoot  
- [ ] Settling to ±x% within target time

**Output**  
CSV + auto-generated plot with annotated droop/settling metrics.

---

# Headline Specs
- **ADC:** Analog Devices **AD4630-24**, up to **2 MSPS**, 24-bit  
- **Channels:** 2 differential, fully-differential amp (FDA) driven  
- **Trigger:** external **TRIG IN** → Schmitt-conditioned → CNV; **Manual Trigger** pad  
- **I/O domains:** **1.8 V** (ADC) and **3.3 V** (MCU) with level translators  
- **Reference:** **LTC6655-5** low-noise, low-drift  
- **Rails:** **LT3042** (+5.4 V), **LT3093** (−5.4 V), **LT1763** (1.8 V & 3.3 V)  
- **Configurable 50 Ω trigger termination** for DDG/coax use (DNP by default)

**Bandwidth envelope (typical build)**
- Usable analog BW: ~**300–600 kHz** (tunable via RCs)
- Practical “clean shape” pulse widths: **≥ 5–10 µs** (2 MSPS sampling)
- **Detect-only minimum pulse:** ~**1–2 µs** (1–2 samples)

---

# Design Highlights (Decisions & Rationale)
- **Anti-alias vs. settling (µs pulses in mind):** modest input Rs (~54 Ω) and small caps (≈1–2.7 nF) around the FDA target a **few-hundred-kHz to ~1 MHz** shaping—enough to see µs edges while improving settling and noise.
- **Low-drift reference path:** **LTC6655-5** with **10 µF + 0.1 µF** close to ADC REF minimizes baseline wander in long captures.
- **Trigger integrity:** External TRIG goes through a **Schmitt buffer + enable/OR logic** to produce a crisp **CNV**; **50 Ω** termination is **optional** for DDG/coax use.
- **Signal integrity at the converter:** **33 Ω series** per ADC input leg for damping; **22–33 Ω** on **CNV** after level translation to tame edges.
- **Clean domain split:** Translators isolate the quiet **1.8 V ADC** domain from **3.3 V MCU**, enabling fast SPI and **BUSY** handshaking.

---

## Trigger Modes
- **External-coax (DDG):** populate 50 Ω at TRIG IN; QC/DDG drives a matched line.  
- **Local-logic:** leave 50 Ω DNP; CNV is driven via translator with a **22–33 Ω** series resistor.

---

## Bring-Up Checklist
- **Power-only:** verify +5.4 V, −5.4 V, +3.3 V, +1.8 V rails; REF ≈ 5.000 V.  
- **CNV source:** confirm 3.3 V at Schmitt output and **1.8 V** at ADC CNV.  
- **Zero-volt short noise:** short IN+ to IN− at SMA; log RMS code spread.  
- **Step/edge:** feed a known step; confirm minimal ringing and proper settling.  
- **Drift:** shorted input, log ≥1 h; expect very low baseline drift.

---

## Safety & Limits
- Keep inputs **within safe range** (use passive dividers when needed).  
- Target content **≤ ~1 MHz** and **µs-scale** pulses.  
- For **ns edges/RF**, use a high-BW DSO/GHz digitizer.

---
