# PS 26185 — UHF Meander Antenna (CST Model)

Conformal helmet antenna array, Smart India Hackathon 2026, Team CRAZAC.
This repo/file documents the UHF (433 MHz target) meander element built in
CST Studio Suite 2026 (Time Domain solver, full license).

## Design overview

A single meandered UHF radiator, fed at one end, over a **small ground
strip** (not a full ground plane) — this specific choice fixed a severe
radiation-efficiency problem an earlier full-ground design had (see
"Design history" below).

- Substrate: LCP, εr = 2.9, tanδ = 0.0025 (Rogers ULTRALAM 3850 datasheet)
- Trace/ground conductor: copper (annealed), PEC in the model
- Feed: single Discrete Edge Port, 50Ω, end-fed at the trace's first point

## Current geometry

Parameter list (CST units: mm / MHz / ns):

| Parameter | Value | Meaning |
|---|---|---|
| x0 | -9 | trace start X |
| y0 | 10 | trace start Y |
| seg | 38 | row length (X) |
| pitch | 6 | row spacing (Y) |
| tw | 2 | trace width |
| cu_t | 0.035 | copper thickness |
| sub_h | 0.5 | substrate thickness |
| er | 2.9 | substrate εr |
| t_and | 0.0025 | substrate tanδ |
| gnd_h | 12 | ground-strip height (Y) |
| bx | 76 | board width (X) |
| by | 76 | board height (Y) |
| dy | 4 | unused in this design (legacy) |
| foam_t | 10 | foam layer thickness (head-loading test) |
| head_t | 10 | head layer thickness (head-loading test) |
| R_a | 181 | outer radius of curvature test cylinder |
| xc | bx/12 | curvature cylinder center X |
| zc | -cu_t-R_a | curvature cylinder center Z |

Rows: **12** (indices 0–11), each row = one horizontal run of length `seg`
connected by a vertical jog of `pitch`.

### Bricks (flat antenna board)

```
SmallGround: PEC
  Xmin = x0-(bx-seg)/2   Xmax = x0+seg+(bx-seg)/2
  Ymin = 0               Ymax = gnd_h+5
  Zmin = -cu_t           Zmax = 0

Substrate: LCP
  Xmin = x0-(bx-seg)/2   Xmax = x0+seg+(bx-seg)/2
  Ymin = 0               Ymax = by+5
  Zmin = 0               Zmax = sub_h
```
**Xmin/Xmax formula keeps the trace centered in the board automatically**
for any `seg`/`bx` — fixed a bug where an earlier `-bx/3, bx/2` formula
left the trace off-center. Y is deliberately asymmetric — the ground
strip only covers the feed end, matching the "ground only near feed"
topology that fixed the efficiency problem. (As of the last check, this
centering fix still needed to be re-applied in the live project — confirm
Xmin/Xmax match the formula above, not `-bx/3, bx/2`, before trusting any
new run.)

### Meander trace polygon (22 points, 12 rows)

```
(x0, y0)
(x0+seg, y0)
(x0+seg, y0+pitch)
(x0, y0+pitch)
(x0, y0+2*pitch)
(x0+seg, y0+2*pitch)
(x0+seg, y0+3*pitch)
(x0, y0+3*pitch)
(x0, y0+4*pitch)
(x0+seg, y0+4*pitch)
(x0+seg, y0+5*pitch)
(x0, y0+5*pitch)
(x0, y0+6*pitch)
(x0+seg, y0+6*pitch)
(x0+seg, y0+7*pitch)
(x0, y0+7*pitch)
(x0, y0+8*pitch)
(x0+seg, y0+8*pitch)
(x0+seg, y0+9*pitch)
(x0, y0+9*pitch)
(x0, y0+10*pitch)
(x0+seg, y0+10*pitch)
(x0+seg, y0+11*pitch)
(x0, y0+11*pitch)
```
Built via Trace From Curve (Thickness=cu_t, Width=tw), then translated to
z=sub_h.

### Port
Discrete Edge Port, 50Ω, S-Parameter type, end-fed at (x0, y0):
`(x0, y0, 0)` to `(x0, y0, sub_h)`, Position = end1.

## Head/foam curvature-loading model (in progress)

Two nested cylinders under the flat antenna board, approximating a curved
head + foam spacer:

```
FoamTube: Foam material
  Orientation: V-axis
  Outer radius = R_a          Inner radius = R_a-foam_t
  Ucenter = xc                Wcenter = zc
  Vmin = 0                    Vmax = by+5

HeadTube: Head material
  Orientation: V-axis
  Outer radius = R_a-foam_t   Inner radius = R_a-foam_t-head_t
  Ucenter = xc                Wcenter = zc
  Vmin = 0                    Vmax = by+5
```

Materials:
- **Foam**: εr = 1.05, tanδ = 0.001 (evaluated 330–480 MHz)
- **Head**: εr = 43.5, σ = 0.87 S/m (standard 450 MHz head-tissue-liquid
  reference values), evaluated 330–480 MHz

**Open caveats on this sub-model:**
- `R_a = 181mm` is roughly double a typical adult head radius (~85–95mm)
  — check whether this was meant as a radius or a diameter before
  trusting curvature-effect results from it.
- `head_t = 10mm` is well under one skin depth in this tissue material at
  433 MHz (~26mm, from σ=0.87 S/m) — a shell this thin may let a
  significant fraction of the field pass through rather than being
  absorbed as a real head would. An earlier version used `head_t=40`,
  closer to (if still slightly under) one skin depth.
- Whether the earlier `Xmin/Xmax` centering fix (see Bricks section
  above) was applied to this specific project (`curve_base`) is
  unconfirmed — no brick screenshot has been taken for this run yet.

**Result — head/foam-loaded run (`curve_base`), same geometry as the flat
design above, now solved:**
- S1,1: **-15 dB dip at ~406 MHz** — much deeper than the flat/unloaded
  design's -6.8dB, and closer to a genuinely good match with no capacitor
  needed
- Z1,1: 76.5-j24.0Ω at 409.6 MHz, 51.6-j50.5Ω at 413.6 MHz — real part
  naturally close to 50Ω
- **Key finding: the head/foam load pulled resonance down by ~54 MHz
  (~12%)**, from the flat design's ~460 MHz to ~406 MHz. This means an
  unloaded (free-space) design tuned to land exactly on 433 MHz would
  likely resonate well below 433 MHz once actually worn near a head —
  the final tuning target should account for this loading shift, not
  just the flat-model resonance.

## Solver settings (current / final-quality run)

- Solver: Time Domain, Hexahedral FIT mesh
- Accuracy: **-30 dB**
- Mesh: back to the fine/locked spec for this run — **20 cells/wavelength,
  60 cells/max model box edge** (coarser 12/30 settings are used only
  during parameter-scouting sweeps, not final runs)
- Acceleration: CPU up to 6 devices, **Hardware acceleration (GPU) up to
  6 devices**, both enabled
- GPU: NVIDIA GeForce RTX 4060 Laptop GPU. This card is **not on CST's
  officially supported GPU list** (only Tesla/Quadro/RTX Pro/A-series are
  officially supported), so it needs an explicit opt-in:
  1. Set a Windows system environment variable:
     `CST_HWACC_ALLOW_UNVERIFIED_HARDWARE = 1`, then restart the PC.
  2. **Before opening CST, verify the GPU is recognized**: run
     `HWAccDiagnostics_AMD64.exe`, found in the CST install folder's
     `AMD64` subfolder (e.g.
     `C:\Program Files\CST Studio Suite 2026\AMD64\HWAccDiagnostics_AMD64.exe`).
     It should report the GPU as detected with status OK before you rely
     on it inside CST.
  3. In CST: **License and Acceleration** dialog → check **Hardware
     acceleration**, set devices ≥1.
  - The Time Domain Solver doesn't need strong double-precision (FP64)
    performance, which is the main weakness of consumer/laptop GPUs, so
    this works well for this specific solver despite being unsupported
    hardware.

## Latest results (flat, unloaded config — before head/foam)

At the 12-row/seg=38/pitch=6 flat design:
- S1,1: -6.8 dB dip at **~460 MHz** (target: 433 MHz, ~6% high)
- Z1,1: real peak ~140Ω near 462 MHz
- Farfield at 433 MHz: directivity **+1.92 dBi**, radiation efficiency
  **-0.345 dB (~92%)**, total efficiency -8.8 dB (mismatch-limited,
  since 433 MHz is off the true resonance)

The ~92% radiation efficiency is the headline result — an earlier
full-ground-plane design (see history) only achieved ~0.43%. See the
head/foam-loaded result above for how this design behaves near a head.

## Design history / key lessons

1. **Full ground plane under the trace kills efficiency.** An initial
   design (200×200mm full PEC ground, 0.5mm below the trace) gave only
   ~0.43% radiation efficiency — the trace's image current in the close
   ground almost cancels it. Switching to a small ground strip under only
   the feed end (this design) fixed it to ~92%.
2. **Tight row spacing (pitch) can backfire.** Reducing pitch from 6mm to
   4mm (to fit more rows in less height) pushed the resonance *higher*,
   not lower, despite more total trace length — adjacent rows only 2mm
   apart start canceling each other's contribution to the effective
   electrical length. Stay at pitch=6 or coarser; this was verified
   empirically, not assumed.
3. **Simple length-vs-frequency scaling is unreliable across large
   changes.** A rough estimate (assuming frequency ∝ 1/length) predicted
   ~424 MHz for the current geometry; the actual result was ~460 MHz.
   Treat any such estimate as a rough starting point only — verify by
   simulation.
4. Trace-from-curve: Thickness field = cu_t, Width field = tw — do not
   swap (swapping builds tall walls instead of a flat ribbon).
5. Delete old trace/curve before rebuilding — stale geometry causes
   "Trace from Curve" to be greyed out or produce mismatched results.
6. A port's z=0 end needs a solid conductor directly beneath it, or you
   get an invalid ~flat, high-impedance result (looks like an open
   circuit, not a resonance).
7. Multiple field monitors and a tight steady-state accuracy limit both
   independently increase simulation time significantly — disable
   monitors and loosen accuracy (e.g. -30dB) while parameter-scouting;
   only use fine mesh/accuracy settings for the final confirmed design.
8. Polygon points must strictly alternate horizontal/vertical moves (no
   diagonal jumps) or the curve self-crosses.
9. GPU acceleration doesn't reduce mesh cell count or time-step count —
   it only speeds up the per-cell, per-step computation. A high-Q
   resonance with a fine mesh can still take a long time even with a GPU;
   mesh/accuracy settings and GPU acceleration are separate, additive
   levers.

## Open items / not yet done

- Resonance is ~460 MHz on the flat design, still ~6% above the 433 MHz
  target — one more row was the planned next nudge.
- No matching network built into the model yet.
- Head/foam curvature-loading run above is built but not yet solved —
  fix the centering bug and reconsider R_a/head_t first (see caveats).
- Not yet curved to the helmet's actual radius as a final design (the
  cylinder above is a test rig, not the final conformal geometry).
- Real helmet mounting-zone dimensions are still unconfirmed (only an
  external estimate of ~80×60mm exists) — current footprint target of
  ~6×6cm (currently 76×76mm) is based on general tactical-helmet
  hardware precedent (NVG counterweight pouches, patch panels), not a
  measured number from the actual helmet.
- 433 MHz itself is the literature-based starting frequency, not yet
  verified against the actual radio hardware's real operating band.
- EBG-backed ground plane and L-band (1.6 GHz) patch are separate,
  not-yet-built parts of the full array — see project overview docs.
