# Examples

Ready-to-run netlists using the flat GF180MCU 3.3 V model. Both work in
**LTspice** and **ngspice**.

| File | What it shows |
|------|---------------|
| `idvd_nmos.cir` | NMOS Id–Vd family (Vgs = 1.1 … 3.3 V) |
| `inverter_vtc.cir` | CMOS inverter DC transfer curve (VTC) |

> **Units:** GF180 is drawn in **meters** — geometries are written like
> `L=0.28e-6 W=1e-6`. To use microns instead, add `.option scale=1u` and write
> `L=0.28 W=1`.

## Run in LTspice
1. `File > Open` the `.cir` file (set file type to "All Files" if needed).
2. `Simulate > Run`.
3. Add a trace: `Id(M1)` for the Id–Vd example, `V(out)` for the inverter.

## Run in ngspice
```
cd examples
ngspice idvd_nmos.cir
```
Then: `run` and `plot -i(Vd)` (Id–Vd) or `plot v(out)` (inverter). A ready-made
`.control` block is included but commented out — uncomment it for automatic
plotting in ngspice batch mode.

## Reminder
Core supply is **3.3 V**. The model is extracted from the L ∈ [0.28, 0.5] µm,
W ∈ [0.5, 1.2] µm bin and stays accurate a good way outside it (see the top-level
`README.md` for the L/W accuracy tables).
