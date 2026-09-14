# 3CatNXT PocketQube — Hardware

Hardware repository for the IEEE GRSS PocketQube built at the [NanoSat Lab (UPC)](https://nanosatlab.upc.edu/).

It holds the complete electrical and mechanical design of the satellite: KiCad projects for every
PCB in the stack, the SolidWorks/STEP assembly of the full spacecraft, the deployment mechanisms,
and the ground/mechanical support equipment used to integrate and test it.

This repository contains **design data only** — there is nothing to build or run. Flight and
ground software live elsewhere, as does the PL3 payload hardware
([nanosatlab/pocat-rfi-5g](https://github.com/nanosatlab/pocat-rfi-5g)).

---

## Repository layout

| Path | What it is |
|---|---|
| `pq_eps/` | Electrical Power Subsystem |
| `pq_obc_comms/` | On-Board Computer + COMMS transceiver |
| `pq_adcs/` | Attitude Determination and Control System |
| `pq_pl1/` | Payload 1 interface board |
| `pq_pl2/` | Payload 2 — L-band receiver |
| `pq_topboard/` | +Z face board |
| `pq_botboard/` | −Z face board |
| `pq_latboard/` | Lateral (side) face board |
| `pq_ymag_board/` | +Y magnetorquer board |
| `pq_botnotsobot/` | Deployment-switch / slider board |
| `pq_egse/` | EGSE — electrical ground support equipment breakout |
| `assembly/` | SolidWorks + STEP mechanical assembly of the whole satellite |
| `Vitrina_PQ/` | Display case (acrylic showcase) for the engineering model |

Each KiCad project directory typically contains the `.kicad_sch` / `.kicad_pcb` / `.kicad_pro`
triplet, an exported `.step` for mechanical integration, and a `production/` folder with the IPC
netlist and (where fabricated) Gerbers. Directories suffixed `_old` are superseded revisions kept
for traceability; `pq_pl2/Imports/` holds the original Altium design that PL2 was migrated from.

---

## The boards

The satellite is a stack of 40 × 40 mm boards held between structural face boards that also carry
the solar cells and thermal sensors.

### Stack boards (40 × 40 mm)

**`pq_eps` — Electrical Power Subsystem** · 4-layer, 1.6 mm

Solar energy harvesting through **SPV1040** MPPT boost converters, battery charging and
power-path management by an **LTC4040**, coulomb counting via a **DS2782** fuel gauge, and
regulated rails from an **ISL9120** buck-boost. Carries the kill-switch logic that gates the
main regulator, plus the antenna burn-wire release (`BURNCOMMS`) and per-rail enables for the
ADCS and payloads. Battery is a single Li-ion **LP14430** cell.

**`pq_obc_comms` — On-Board Computer & COMMS** · 4-layer, 1.6 mm

**STM32L476RGT6** microcontroller with a 32 MHz TCXO and a 32.768 kHz RTC crystal, paired with a
**SX1262** sub-GHz LoRa transceiver. RF front end uses a **BGS12PL6** antenna switch out to U.FL
and MS-156C test connectors. The STM32CubeMX pin configuration lives alongside the project in
`STM32L476RG_config/STM32L476RG_config.ioc`, and an interactive BOM is generated at
`pq_obc_comms/bom/ibom.html`.

**`pq_adcs` — Attitude Determination & Control** · 6-layer, 1.6 mm

Sensing from an **IIM-42652** 6-axis IMU and an **MMC5983MA** magnetometer, with sun-sensor
conditioning through **LPV542** op-amps and a **TMUX1108** analog multiplexer.

Two revisions are tracked:
- `PQ_ADCS/` — original design, magnetorquer coils driven by a **BD2606MVV**.
- `PQ_ADCS_HBridge/` — current revision, redesigned around **DRV8214** H-bridge drivers. The
  vendor-supplied symbol, footprint and STEP for the DRV8214 are vendored in
  `PQ_ADCS_HBridge/DRV8214RTER/`.

**`pq_pl2` — L-band receiver payload** · 4-layer, 1.58 mm

Full receive chain: **BFCG162W** LNA → **LEE2-6+** gain stage → **MAX2121** direct-conversion
tuner, with an **LT5537** RF power detector for level monitoring and an **LT3048** ultra-low-noise
LDO feeding the RF section. Clocked by an **ECS-TXO-2016MV** TCXO; MS-156C and U.FL connectors
provide antenna and test access.

**`pq_pl1` — Payload 1 interface** · 2-layer, 1.6 mm

Passive interface/adapter board exposing the stack connector to the PL1 experiment.

### Face and structural boards

**`pq_topboard` / `pq_botboard` / `pq_latboard`** · 2-, 6- and 6-layer respectively

These form the outer faces of the cube. They mount **SLCD-61N8** solar cells and **TCN75A**
digital temperature sensors, and route the harness into the stack. The bottom board additionally
carries **TPS7A7001/7002** LDOs, and the lateral board brings out coaxial connectors for the
deployable antenna. Face boards are 2 mm thick because they are structural.

**`pq_ymag_board`** · 6-layer, 2 mm, 45 × 45 mm

+Y magnetorquer — an air-core coil etched into the PCB, driven from the ADCS board.

**`pq_botnotsobot`** · 2-layer, 1.6 mm, 64 × 58 mm

Slider board carrying the **D2FLA** deployment/kill switches that hold the satellite inert inside
the deployer.

**`pq_egse`** · 6-layer

Electrical Ground Support Equipment. A large breakout fixture that fans every stack connector out
to 0.1" headers so subsystems can be probed, powered and stimulated individually on the bench.
Fabrication Gerbers and drill files are checked in under `pq_egse/manufacturing/`.

---

## Mechanical assembly

`assembly/` is the SolidWorks model of the integrated spacecraft. The top-level assemblies are:

- `ieee_grss_pocketqube.SLDASM` — the full satellite.
- `ieee_grss_pocketqube_beta antena.SLDASM` — variant with the deployed antenna configuration.

Supporting content is organised by subsystem — `eps/`, `adcs/`, `obc&comms/`, `payloads/`,
`structure/`, `lateral/`, `bottom/`, `slider/` — and each PCB folder carries the board as both a
`.step` and a `.SLDPRT`, together with the Edge_Cuts and silkscreen DXF/SVG exports used to keep
the mechanical model in sync with the electrical design.

Of note:

- **`assembly/lateral/antena-deployment/`** — the COMMS tape-spring antenna and its burn-wire
  release: turret, pivot, spring, clip and the resistor that cuts the restraint.
- **`assembly/Lband_Deployment_Mechanism/`** — deployment mechanism for the L-band payload
  antenna (`Lband3d_2res_v7.2.SLDASM`), with its own board and antenna sub-assemblies and a
  `STEP_files/` export set.
- **`assembly/payloads/pl3/`** — RFI-5G payload structure (antenna, supports, SMP interface).
  Mechanical only — the PL3 electrical design lives in its own repository,
  [nanosatlab/pocat-rfi-5g](https://github.com/nanosatlab/pocat-rfi-5g).
- **`assembly/MGSE_Assembly/`** — Mechanical Ground Support Equipment: the jigs and supports used
  to hold the satellite during integration.
- **`assembly/structure/`** — chassis parts plus the screw, nut and spacer library (M2/M3 ISO).

---

## Working with this repository

**Tools.** KiCad 7 or newer is required; the EPS and ADCS H-Bridge projects have been saved with a
more recent file format (`20260206`) than the rest of the stack, so opening them in an older KiCad
will fail. Mechanical files are SolidWorks, with STEP exports provided for anyone without it.

**Exports are checked in on purpose.** STEP files, Edge_Cuts/silkscreen DXFs and `production/`
netlists are tracked so that mechanical integration and manufacturing can proceed without
re-running KiCad. If you change a board outline or a connector position, re-export the STEP and
DXF and commit them in the same change — otherwise the mechanical assembly silently drifts out of
date.

**What is ignored.** `.gitignore` excludes KiCad backups, autosaves, lock files, netlists, and
generated BOM `.csv`/`.xml`. Note that it also ignores `production/` and `*-backups/` directories
by default; the ones present in the repository were committed before those rules were added and
remain tracked.

**Branching.** Work happens on per-subsystem branches (`eps_updated`, `adcs_hbridge_revision`,
`Lateral_and_bottom_connectors`, …) which are merged into `main` once the revision passes review.
Binary CAD files do not merge — coordinate before two people edit the same board.

---

## Status

The EPS and ADCS H-Bridge revisions have completed their design review. PL2 has been migrated off
Altium and routed. PL3 appears here as a mechanical model only; its electronics are developed in
[nanosatlab/pocat-rfi-5g](https://github.com/nanosatlab/pocat-rfi-5g). See the git history for the
revision trail of each subsystem.
