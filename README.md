# NanoPulse - A Precision 24-bit Pulse Capture Digitizer
Hardware project, the firmware will be in another repo.

# Purpose & Use Cases

**NanoPulse** is a 2-channel, 24-bit, trigger-synchronized digitizer for **µs-scale pulses**. It delivers a cleaner, more deterministic “scope view” with higher vertical resolution, lower drift, and easy automation.

---


## Headline Specs
- **ADC:** Analog Devices **AD4630-24**, up to **2 MSPS**, 24-bit  
- **Channels:** 2 differential, fully-differential amp (FDA) driven  
- **Trigger:** external **TRIG IN** → Schmitt-conditioned → CNV; **Manual Trigger** pad on board  
- **I/O domains:** **1.8 V** (ADC) and **3.3 V** (MCU) with level translators  
- **Reference:** **LTC6655-5** low-noise, low-drift  
- **Rails:** **LT3042** (+5.4 V), **LT3093** (−5.4 V), **LT1763** (1.8 V & 3.3 V)  
- **Configurable 50 Ω trigger termination** for DDG/coax use (DNP by default)

**Bandwidth envelope (typical build):**
- Usable analog BW: ~**300–600 kHz** (tunable with RCs)
- Practical “clean shape” pulse widths: **≥ 5–10 µs** (2 MSPS sampling)

---


## 1) Scope-replacement for pulsed benches (µs pulses)

**What it solves**  
Verify the **actual** applied pulse (rise/fall, overshoot, flatness, droop) with a deterministic external trigger—without wheeling in a DSO.

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

> **Why this exists:** Many pulse-test tasks don’t need GHz bandwidth—but they **do** need **deterministic trigger, high vertical resolution, and low drift**. NanoPulse focuses on that window (hundreds of kHz, µs pulses), giving cleaner data and simpler automation than a general-purpose scope.

> **Safety/Range:** Keep inputs within the board’s safe range (use simple passive dividers when needed). 50 Ω termination is configurable for external DDG triggers.
