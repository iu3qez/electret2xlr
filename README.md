# electret2xlr

Active adapter that powers an electret microphone from P48 phantom power and presents its signal to a balanced XLR input — no external supply. Built as pcb-as-code with the [`pcb`](https://github.com/diodeinc/pcb) toolchain (Zener language). Designed for a ham-radio shack, so **RF immunity is a first-class requirement**, not an afterthought.

The electronics live in an external **copper-foil enclosure** (Faraday shield) and connect to the sound card through a shielded cable ending in a **flying male XLR plug** (e.g. Neutrik NC3MX).

## How it works

Three stages (~15 passives/actives plus connectors):

1. **Supply (capacitance multiplier).** Phantom 48 V is tapped symmetrically from XLR pins 2/3. An 8.2 V zener (D1) sets a raw reference; R9 47 kΩ + C4 47 µF push the corner below 1 Hz to scrub the zener's avalanche noise; NPN follower Q1 (MMBT3904) buffers it into a clean **VPIP ≈ 7.5 V** rail. Capsule bias comes off VPIP through R10 6.8 kΩ.
2. **Audio (matched class-A followers).** Two PNP emitter followers (MMBT3906): **Q2** (hot) takes the capsule signal (AC-coupled via C2) and modulates the current drawn from pin 2 through R1 7.5 kΩ; **Q3** (cold) is identical but AC-grounded, drawing from pin 3 through R2 10 kΩ. Result: pseudo-balanced ~78 Ω source, ~3 mA/pin symmetric. CB2/CB3 emitter-bypass caps are **1 µF** (not 22 µF — a larger value pushes the HP knee below 1 Hz and defeats the intended ~30 Hz roll-off).
3. **Input.** Capsule solder pads (MIC1) and an optional 3.5 mm TRS jack (J2), preceded by a π anti-RF filter (CIN 220 pF + FB1 ferrite + RIN 100 Ω) — the mic cable is the main antenna, so this is the front line against HF detection on the capsule FET.

Extra RF defenses: CP2/CP3 (100 pF C0G, pin2/pin3→GND), CSH (1 nF, shell→GND), and a 4-layer board with internal ground planes in addition to the copper enclosure.

## Key figures (SPICE-verified)

From `docs/sim-results.md` (ngspice via `pcb sim`):

| Quantity | Simulated | Target |
|---|---|---|
| V(VPIP) | 7.49 V | 7.0–8.0 V |
| V(MICOUT) | 4.09 V | 3–5 V |
| Current per pin (P2 / P3) | 3.00 / 3.13 mA | 2.5–3.5 mA |
| P2 vs P3 imbalance | 4.4 % | < 10 % |
| Flatness 100 Hz–20 kHz | 0.44 dB | ±1 dB |
| Low −3 dB roll-off | ≈ 30.6 Hz | 20–60 Hz |

The differential output phase at 1 kHz is ≈ −179° (inverted vs the conventional sign) — a known trait shared with the reference design, left as a bench-verification item, not fixed here.

## Build

Prerequisites: **`pcb`** (Rust binary, not on PyPI) and **KiCad 10.x** (only needed for `pcb layout`).

```bash
curl -fsSL https://raw.githubusercontent.com/diodeinc/pcb/main/install.sh | bash
export PATH="$HOME/.local/bin:$PATH"        # pcbc 0.4.7 used here
git clone https://github.com/iu3qez/electret2xlr && cd electret2xlr
pcb build electret2xlr.zen                  # 32 components, no errors
pcb layout electret2xlr.zen                 # open/update layout in KiCad
```

Footprints and symbols are **vendored** (`footprints/`, `symbols/`), so a clean clone builds offline — **no `pcb auth login` or online registry needed**. KiCad 10.x install: official AppImage (no sudo), Flatpak `org.kicad.KiCad`, or a distro package with 10.x.

**Placement and routing are left to the user.** The imported `layout/layout.kicad_pcb` (4-layer, 1.6 mm, provisional 30×40 mm outline) has components at netlist-import coordinates. DRC therefore reports the 51 unrouted nets plus a few overlap violations from `J1` auto-dropped onto the C4/Q2/J2 cluster — both clear once parts are placed.

Optional SPICE re-run (needs `ngspice`):

```bash
NGSPICE=/path/to/ngspice pcb sim electret2xlr.zen --config sim_mode=true --setup testbench/setup_dc_ac.cir -v
```

`sim_mode=true` drops the two capsule test points (no stdlib SPICE model); a normal `pcb build` keeps them as real pads.

## Wiring

- **J1 → flying XLR** (`footprints/J1_XLR3_WirePads.kicad_mod`, 3 THT pads): pin1 = GND/shield, pin2 = hot (P2), pin3 = cold (P3). Solder the shielded cable here; wire the other end to a male XLR plug (pin1 shield, pin2 +, pin3 −). Bond the copper enclosure to GND at pin 1.
- **Capsule** on MIC1 pads (2.54 mm pitch): signal/bias to one pad, GND to the other; observe the capsule's own polarity.
- **Jack J2** (optional, on the enclosure): tip = signal/bias, ring bridged to tip, sleeve = GND. Smartphone TRRS headsets need an external TRRS→TRS adapter.
- **SHELL pad:** the copper-foil contact (net `SHELL`, where CSH lands) is a hand-drawn copper feature tied to GND, not a netlist part — so `pcb build` emits an expected single-pin-net ERC warning for `SHELL`.

## Attribution & license

Topology inspired by [tphakala/p48-pip-adapter](https://github.com/tphakala/p48-pip-adapter). That project's CC BY-NC 4.0 covers its concrete design (layout, gerbers, docs), not the circuit topology — a widely published functional idea. This repo's Zener source, layout, and docs are written from scratch, so no license constraint is inherited; attribution here is a courtesy.

## Reference docs

- `docs/superpowers/specs/2026-07-15-electret2xlr-design.md` — full design spec
- `docs/sim-results.md` — SPICE verification (DC + AC)
- `docs/meccanica.md` — mechanical fit study (historical: in-connector mount, superseded by the external enclosure)
- `docs/bom.md` — bill of materials
