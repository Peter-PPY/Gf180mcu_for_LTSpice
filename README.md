# GF180MCU 3.3 V Flat BSIM4 Model for LTspice (PTM-style)

A single, flat `.model` card for the GlobalFoundries **GF180MCU** 3.3 V core
NMOS and PMOS devices — distilled from the official open PDK into a PTM-style,
drop-in model for **LTspice**, **ngspice**, or **HSPICE**, without the PDK's
subcircuit / binning / statistical wrapper.

> **TL;DR** — Drop-in 3.3 V NMOS/PMOS for a real, fabbable 180 nm process.
> Accurate across a useful geometry range (≈ 0 % inside the fit bin, ~2 % out
> to L = 2 µm / W = 5 µm). **Not** a foundry sign-off model.

This is the GF180 companion to the SKY130 flat model — two real, tape-out-capable
open PDKs made easy to explore in LTspice, for teaching and first-pass design.

---

## What this is

The real GF180 device models (`nmos_3p3`, `pmos_3p3`) ship as `.subckt` wrappers
around **binned**, **corner-based**, **statistically-parameterised** BSIM4 cards.
This library collapses that into one flat BSIM4 (level 54) card per device:

| Setting            | Value                                          |
|--------------------|------------------------------------------------|
| Corner             | typical (`t`)                                  |
| Statistics / MM    | **off** (global + mismatch = 0)                |
| Frozen fit bin     | **L ∈ [0.28, 0.5] µm, W ∈ [0.5, 1.2] µm** (bin 4) |
| Model type         | BSIM4, `level = 54`, `version = 4.5`           |
| Units              | SI / **meters** (native GF180)                 |

Crucially, the card **keeps the in-bin length/width correction terms**
(`binunit = 2`), so it reproduces the real device *across the whole fit bin*, not
just at one point.

**Model names**

- `gf180_nmos_3p3` — 3.3 V NMOS, `level=54`
- `gf180_pmos_3p3` — 3.3 V PMOS, `level=54`

---

## How to use

GF180 is drawn in **meters** (native). Two equivalent styles:

```spice
.include gf180_ptm_style.lib

* Style A -- native meters:
M1 nd ng ns nb gf180_nmos_3p3 L=0.28e-6 W=1e-6
M2 pd pg ps pb gf180_pmos_3p3 L=0.28e-6 W=1e-6

* Style B -- microns via scale (same result):
* .option scale=1u
* M1 nd ng ns nb gf180_nmos_3p3 L=0.28 W=1
```

Notes:
- Call the devices with **`M`** (plain `.model`), not `X`.
- Core supply is **3.3 V** (not 1.8 V like SKY130).
- LTspice maps `level=54` to BSIM4.

---

## Validation

Checked against the **real GF180 binned model** (`nmos_3p3` / `pmos_3p3`, with
the simulator doing its own bin selection) in **ngspice 42**, statistics off.

| Test (vs. real GF180 device) | NMOS | PMOS |
|------------------------------|------|------|
| Id–Vd sweep, max error inside fit bin (335 pts) | **0.00 %** | **0.00 %** |
| Id at Vgs = Vds = 3.3 V (L=0.28 µm, W=1 µm) | 520.3 µA (exact) | 248.1 µA (exact) |

Because the flat card retains the binning corrections, it matches the real
device **exactly at every geometry inside the fit bin** — e.g. (L, W) =
(0.28, 1), (0.4, 0.8), (0.5, 1.2) µm all give 0.00 % error.

> **Important:** validated in **ngspice**, not LTspice. LTspice's BSIM4 engine
> differs slightly, so spot-check one Id–Vd curve in LTspice before relying on
> absolute numbers. LTspice's BSIM4 also has known quirks (e.g. Cgs/Cgd not
> echoed in the `.op` log); DC / transient is generally fine.

---

## Limitations (please read)

This is a single-corner, single-bin extraction. It is exact inside the fit bin
and degrades gracefully outside it (measured vs. the real device, Vgs=Vds=3.3 V):

**Channel length (L), W = 1 µm** — fit bin L ∈ [0.28, 0.5] µm:

| L | 0.28 µm | 0.5 µm | 0.7 µm | 1 µm | 2 µm |
|---|---------|--------|--------|------|------|
| Error | 0 % | 0 % | ~0.2 % | ~1.8 % | ~1.9 % |

**Channel width (W), L = 0.28 µm** — fit bin W ∈ [0.5, 1.2] µm:

| W | 0.3 µm | 0.5 µm | 1.2 µm | 2 µm | 5 µm |
|---|--------|--------|--------|------|------|
| Error | ~3.5 % | 0 % | 0 % | ~0.7 % | ~1.2 % |

→ **Usable across a wide range (~2 % out to L = 2 µm, W = 5 µm).** Largest error
is at very narrow devices (W < 0.5 µm, ~3–4 %). For minimum-width devices in
precision analog, re-extract from the matching bin.

**Other constraints**

- **Typical corner only** — no fast/slow (`f`/`s`/`fs`/`sf`).
- **No statistics** — no mismatch, no Monte-Carlo.
- Only the **3.3 V** core devices — not the 5 V / 6 V or native-Vt variants.
- Subcircuit-level parasitics outside the intrinsic BSIM4 device (e.g. SAB
  series resistance) are not reproduced.
- **Not for sign-off.** For verification or tape-out, use the full
  `gf180mcu_fd_pr` PDK models with a supported simulator.

---

## Regenerating for other geometries / corners / devices

The flattening is deterministic. A card can be re-extracted for a different bin,
corner (fast/slow), or device family (5 V / 6 V), restoring ~0 % accuracy at that
new operating region. For a fully scalable model, use the real binned PDK
subcircuit.

---

## Source, license, and attribution

This library is a **derived work** of the GlobalFoundries GF180MCU Open Source PDK.

- **Upstream source:** `gf180mcu_fd_pr` primitive device models
  — https://github.com/google/gf180mcu-pdk
  (models: https://github.com/google/globalfoundries-pdk-libs-gf180mcu_fd_pr)
- **Upstream license:** **Apache License 2.0**
- Copyright © GlobalFoundries / Google LLC and PDK contributors.

Redistribution complies with Apache-2.0. This repository:

1. includes the full `LICENSE` (Apache-2.0),
2. retains this attribution notice, and
3. **states the change made:** the original binned/statistical `.subckt` models
   were collapsed to a flat BSIM4 `.model` card at the typical corner, fit bin
   L ∈ [0.28, 0.5] µm × W ∈ [0.5, 1.2] µm, statistics disabled.

Extraction tooling and documentation by **VirtuChip.AI**. No warranty; see
*Limitations* and the Apache-2.0 disclaimer.

---

## Disclaimer

Provided **as-is** for education and preliminary design. It is **not** an
official GlobalFoundries / foundry model and carries no accuracy guarantee for
tape-out. Always validate against the official PDK before committing silicon.
