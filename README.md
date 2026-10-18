# Naspier Core

**A Pixhawk v6X-compatible flight controller system — design study, revision REV0 VER0
(started September 2026), referred to below as V0.**

Naspier Core started as a single autopilot core (FMU) module built around the **STM32H743IIK6**. It
is now a **three-board system**, designed and documented together:

| Board | What it is | Links to |
|---|---|---|
| **Naspier Core** (core module) | The FMU: H743, power tree, storage, IMU1 + baro 1 | Carrier over the standard Pixhawk **X1 (100-pin) + X2 (50-pin)** Hirose DF40 connectors; IMU board over the 34-pin **X3** |
| **Naspier IMU V2** (sensor board) | Second and third IMU, baro 2, magnetometer, heater, calibration EEPROM | Core X3, through a 34-pin flex cable |
| **Naspier PAB carrier** (base board) | Power mux, IO MCU, Ethernet PHY, CAN, all vehicle connectors and PWM headers | Core X1 + X2 receptacles, built to the Pixhawk Autopilot Bus standard |

The core still follows the v6X interface on its own, so it is meant to drop onto any
**Pixhawk v6X-standard carrier**, and the carrier is meant to accept standard v5X / v6X modules.

| Top | Bottom |
|---|---|
| ![All three boards, top](docs/images/rendered-system-top.png) | ![All three boards, bottom](docs/images/rendered-system-bottom.png) |

*3D renders, October 2026: core module (top left), IMU V2 board, PAB carrier (bottom). The carrier
layout is at an early placement stage.*

---

## ⚠️ Please read this first

**This is a design project, not a product, and not an officially used board.**

- It is a personal hardware design exercise. None of the boards has been fabricated, assembled,
  bring-up tested, flight tested, certified, or used in any real vehicle.
- It is **not** an official Pixhawk / Dronecode / PX4 reference design, and it is not endorsed by,
  affiliated with, or reviewed by any of those projects.
- Nothing here is production-ready. Treat every number in this repository as *intent*, not as a
  verified result. **Do not fly it.**
- If you build from these files, you do so entirely at your own risk. See [LICENSE](LICENSE) —
  the design is provided **as is, with no warranty of any kind**.

## What is published here, and what is not

This repository is **open documentation, not open-source hardware.** It will grow as the design
progresses, but it has a deliberate boundary:

| | Status |
|---|---|
| Schematic (PDF) — core module, IMU V2 board, PAB carrier, flex cable | ✅ Published now |
| System block diagram (preliminary), connector pinouts, dimensions, 3D renders | ✅ Published now |
| **PCB layer images** — layer-by-layer plots | 🕓 Will be published once the PCB layout is finished |
| **Bill of materials** | 🕓 Will be published once the PCB layout is finished |
| **Source files** — KiCad project, schematic, board, libraries | ❌ **Will not be published** |
| **Production files** — Gerbers, drill, pick-and-place, paste/stencil, assembly data | ❌ **Will not be published** |

Enough to read, study, learn from and review the design — not enough to press "order" on it. That
is intentional, and it is not going to change for this revision.

## Why H743 and not H753?

Pixhawk v6X flight controllers ship with the **STM32H753** (DS-012 itself only calls for an STM32H7).
The H753's distinguishing feature over the H743 is its on-chip **cryptographic / hash accelerator**. Naspier Core deliberately uses the
**STM32H743IIK6** instead.

The two parts are pin-compatible in the same UFBGA-201 10×10 package and otherwise share the same
core, clocks, memory map and peripheral set — Cortex-M7 @ 480 MHz, 2 MB flash, 1 MB RAM. Dropping
the crypto block is intentional: **this board is being designed toward a different target than a
standard v6X FMU**, and the secure-element/crypto path is not part of that target. The H743 keeps
the design simpler and cheaper while leaving the v6X electrical interface untouched.

**Supply is the other reason.** H753 stock has been thin, and when distributor inventory runs out
the factory lead times are long. Using the H743 means this board **does not depend on H753
availability**: if H753 stock is gone, the build carries on with the H743 and simply goes without
the crypto accelerator. Nothing else about the board changes.

### ⚠️ If you need FMUv6X's security features, this is the wrong board — use an H753-based module.

---

## Core module

### Board-to-board connectors

X1 and X2 follow the v6X / PAB pinout. Full tables: **[docs/pinout.md](docs/pinout.md)**.

| Connector | Part | Contents |
|---|---|---|
| **X1 — 100P** | Hirose DF40C-100DP-0.4V | Power, PWM 1–8, I2C, CAN, UART, USB, SWD, ADC |
| **X2 — 50P** | Hirose DF40C-50DP-0.4V | Ethernet (RMII), SPI6. The v6X-RT extra pins are left unconnected. |
| **X3 — 34P** | Hirose BM20B-0.8-34DP-0.4V | Link to the IMU V2 board. Not used by the carrier. |

PWM 9–12 and CAN3 (v6X-RT additions) are not available on this revision.

X1 and X2 use the **official Pixhawk PAB footprints** (the Pixhawk mechanical-interface library that
DS-010 points to) on both the core and the carrier, so the pins land where a standard Pixhawk
carrier or module expects them.

**Pin map.** The H743 pin map follows the as-built v6X map used by PX4 `fmu-v6x` and ArduPilot
`Pixhawk6X`, not the pin table in DS-012 rev 0.4.0. The X1 / X2 connector pinouts are the same in both.

### Processor

| | |
|---|---|
| MCU | STM32H743IIK6, ARM Cortex-M7, 480 MHz |
| Package | UFBGA-201, 10 × 10 mm, 0.65 mm pitch |
| Flash / RAM | 2 MB (2M × 8) / 1 MB |
| HS oscillator | 16.000 MHz |
| LS oscillator | 32.768 kHz  |
| RTC backup | MS621FE-FL11E rechargeable coin cell on the VBAT line; charge path (D1, R3) is do-not-place in this revision |
| Status LEDs | Discrete red / green / blue |

### Power structure

Input is **5 V from the carrier** (`VDD_5V_SYS`, four pins on X1). Everything downstream is split so
that no single rail takes the whole board down, and so each sensor domain can be power-cycled
independently — the standard requirement for autopilot sensor recovery.

> **Note — why L1 is there.** L1, a 1 µH inductor on the `VDD_5V_SYS` input, is placed to make the input rail more robust. Many
> off-the-shelf BEC modules do **not** hold their output voltage steady under hard conditions —
> aggressive manoeuvring, heat, high current draw and so on — and sag below nominal, for example
> 5.0 V dropping to 4.6 V on under-peaks. L1 is there only to slow and filter that ripple.
>
> It is the **only** part added on the input power path. Selecting and protecting the input sources
> is the carrier's job: its **LTC4417 power mux** (see [PAB carrier](#pab-carrier-board)).

Design points:

- **Every sensor rail is independently switched** — `VDD_3V3_SENSORS1_EN` … `SENSORS4_EN` and
  `VDD_3V3_SD_CARD_EN` — so firmware can hard-reset a hung sensor or the SD card.
- **Every sensor rail is independently monitored** — each LDL212 output goes through a 10 kΩ / 10 kΩ
  divider to `SCALED_VDD_3V3_SENSORS1..4` and back into the MCU's ADC.
- **Carrier power control and status on X1**, seen from the FMU:
  - Outputs: `VDD_5V_PERIPH_nEN`, `VDD_5V_HIPOWER_nEN`, `VDD_3V3_SPEKTRUM_POWER_EN`.
  - Inputs: `VDD_5V_PERIPH_nOC`, `VDD_5V_HIPOWER_nOC` (over-current flags), and
    `nPOWER_IN_A/B/C` — the active-low "input valid" status of the carrier's power mux. The mux
    chooses the source itself; these are not select lines.
  - `ADC1_3V3` / `ADC1_6V6` are general-purpose analog inputs (3.3 V and 6.6 V full scale), not
    power-module sensing. Battery voltage and current come from digital power modules on I2C1 / I2C2.

### Interfaces exposed to the carrier

- **PWM / timer outputs:** `FMU_CH1`–`FMU_CH8` (X1)
- **Serial:**

  | UART | Use | Flow control |
  |---|---|---|
  | UART7 | TELEM1 | RTS / CTS |
  | UART5 | TELEM2 | RTS / CTS |
  | USART2 | TELEM3 | RTS / CTS |
  | USART1 | GPS1 | — |
  | UART8 | GPS2 | — |
  | UART4 | UART4 (with I2C3 on the carrier) | — |
  | USART3 | Debug console | — |
  | USART6 | Link to the carrier's IO MCU (RC input only on carriers without an IO MCU) | — |

- **CAN:** CAN1, CAN2 (X1)
- **Ethernet:** 100BASE-T RMII — MDIO, MDC, REF_CLK, CRS_DV, RXD0/1, TXD0/1, TX_EN, ETH_POWER_EN
  (the PHY lives on the carrier)
- **I2C:** I2C1 (GPS1 mag/LED/PM1), I2C2 (GPS2 mag/LED/PM2), I2C3 (on-board baro/EEPROM, external).
  I2C4 goes only to the IMU board over X3, not to the carrier.
- **USB:** `USB_D_P` / `USB_D_N` + `VBUS_SENSE`
- **SPI6:** external bus — SCK / MISO / MOSI, two chip selects, reset, two data-ready lines
- **Debug:** `FMU_SWDIO` / `FMU_SWCLK` routed to X1. The board also carries a debug test-point
  group.
- **Other:** buzzer, safety switch + LED, `nARMED`, `FMU_nRST`, `FMU_CAP1`, `FMU_PPM_INPUT`,
  `SPIX_SYNC`, hardware-revision sense lines, RGB status LEDs (discrete red / green / blue)

---

## Sensors and storage

| Board | Part | Function | Bus | VDD bus |
|---|---|---|---|---|
| Core | ICM-45686 | IMU1 | SPI1 | `VDD_SENSOR_BUS_1` |
| Core | MS5611 | Barometer 1 | I2C3 | `VDD_SENSOR_BUS_1` |
| Core | AT24C64D | EEPROM (0x50) | I2C3 | `VDD_SENSOR_BUS_1` |
| Core | FM25V02A | FRAM | SPI5 | `VDD_3V3_AUX` |
| Core | 503398-1892 | microSD | SDMMC2 | `VDD_SD_CARD` |
| IMU V2 | LSM6DSV32X | IMU2 | SPI2 | `VDD_SENSOR_BUS_2` |
| IMU V2 | BMI088 | IMU3 | SPI3 | `VDD_SENSOR_BUS_3` |
| IMU V2 | BMP581 | Barometer 2 (0x46) | I2C4 | `VDD_SENSOR_BUS_4` |
| IMU V2 | RM3100 | Magnetometer (0x20) | I2C4 | `VDD_SENSOR_BUS_4` |
| IMU V2 | AT24C64D | Calibration EEPROM (0x50) | I2C4 | `VDD_SENSOR_BUS_4` |
| IMU V2 | AO3400A | Heater driver | — | `VDD_5V_SYS` |
| Carrier | AT24C64D | Base EEPROM (0x51) | I2C3 | `VDD_3V3_AUX` |

The sensor-to-bus and sensor-to-rail assignment follows the DS-012 sensor-domain layout: one switched
rail per sensor domain, and the IMU-board devices on their own buses. The full assignment is still in
review (see [status](#status-and-roadmap)).

## Naspier IMU V2 board

A small sensor board that sits on its own mount, separate from the core, so the IMUs can be
vibration-isolated and heated.

- **Parts:** LSM6DSV32X, BMI088, BMP581, RM3100 magnetometer (13101 ASIC + 13104 / 13156 coils),
  AT24C64D calibration EEPROM, AO3400A low-side heater driver with resistive loads.
- **Link:** a 34-pin Hirose BM20B receptacle, connected to the core's X3 through a flex cable with
  BM20B receptacles at both ends (drawn on its own schematic page). X3 follows the DS-012 IMU
  connector pinout, except pins 11 and 13, which carry the BMP581 INT and RM3100 DRDY.

## PAB carrier board

A base board built to the **Pixhawk Autopilot Bus** standard
([DS-010](https://github.com/pixhawk/Pixhawk-Standards/blob/master/DS-010%20Pixhawk%20Autopilot%20Bus%20Standard.pdf)),
drawn after the PAB base reference (RC05) and redrawn by hand. It takes the core on DF40HC(3.0)
receptacles (3 mm stack height) and is meant to accept standard v5X / v6X modules as well.

**Power**

- **LTC4417 power mux** choosing between two power bricks (Molex 6-pin) and USB VBUS, output
  `VDD_5V_SYS` to the core.
- **Two BQ24315 protection switches** for the `VDD_5V_PERIPHERAL` and `VDD_5V_HIGHPOWER` rails, with
  enable and over-current flags back to the FMU.
- MCP1727 for the IO MCU rail (`VDD_3V3_IO`); LDL212s for the Ethernet PHY and the Spektrum
  receiver supply.

**Electronics**

- **IO MCU:** STM32F103C8, linked to the FMU over USART6; drives PWM IO CH1–8 and decodes RC input
  (PPM, S.BUS with a 74LVC2G240 buffer, DSM / Spektrum).
- **Ethernet:** LAN8742AI PHY with common-mode chokes.
- **CAN:** two TJA1051 transceivers with termination.
- Reset-request gate on `FMU_nRST`, `VBUS_SENSE` gate, buzzer driver.

**Connectors**

| Port | Connector |
|---|---|
| TELEM1, TELEM2, TELEM3 | JST GH 6-pin, with RTS / CTS |
| GPS1 (+ safety switch, buzzer) / GPS2 | JST GH 10-pin / 6-pin |
| UART4 + I2C3, I2C3 | JST GH 6-pin, 4-pin |
| CAN1, CAN2 | JST GH 4-pin |
| AD&IO, external SPI6 | JST GH 8-pin, 11-pin |
| RC IN, DSM / Spektrum | 2×3 header + JST GH 5-pin, JST PH 3-pin |
| Ethernet | JST GH 4-pin |
| USB | USB-C and JST GH 4-pin |
| FMU debug, IO debug | JST SH 10-pin |
| Power bricks 1 and 2 | Molex 6-pin |
| PWM outputs | Two 3×8 right-angle headers: FMU CH1–8 and IO CH1–8 on `VDD_SERVO` |

**State:** schematic drafted and under review; PCB placement has started (see the renders above).
PWM outputs are not buffered or level-shifted on this revision.

---

## PCB and fabrication — core module

| | |
|---|---|
| Outline | 29.00 × 36.00 mm |
| Stack-up | **12-layer FR4**, 0.5 Oz / 1 Oz copper, ≈**1.6 mm** finished thickness |
| Soldermask | **Blue**, top and bottom, vias tented |
| Mounting | Four corner holes, **2.3 mm drill / 4.0 mm pad**, on a **31.4 × 24.4 mm** pattern |
| Vias | BGA fanout **0.15 / 0.32 mm** (drill / diameter), filled and capped · all other signal vias **0.2 / 0.42 mm** · all through-hole, no microvias |
| Connectors | X1 DF40C-100DP-0.4V(58) · X2 DF40C-50DP-0.4V(58) · X3 BM20B-0.8-34DP-0.4V-51 |
| EDA tool | KiCad 10 |
| Fabrication Target | JLCPCB, NextPCB |

**Component placement follows DS-012.** The board outline, the positions of X1 and X2, the mounting
holes and the keep-out zones are taken from the positioning given in
**[DS-012 — Pixhawk Autopilot v6X Standard](https://github.com/pixhawk/Pixhawk-Standards/blob/master/DS-012%20Pixhawk%20Autopilot%20v6X%20Standard.pdf)**,
not chosen freely. That is what makes the module mechanically as well as electrically droppable onto
a v6X carrier: the connectors land where a v6X carrier expects them, and the mounting holes line up
with the carrier's standoffs. Everything else on the board is placed around those fixed points.

The one piece not governed by DS-012 is **X3**, the 34-pin connector to the IMU V2 board — the
standard does not define its position, so it is ours to choose, constrained by the IMU board stack
rather than by the carrier.

**Impedance control.** The RMII Ethernet group is routed inside an impedance-controlled zone
targeting **50 Ω single-ended**, marked as such on the connectors sheet. Digital and power routing
are kept on their own layers, away from that zone.

**Component placement was revised for signal routing.** Components were moved to shorten and
untangle the signal paths. The STM32H743 is now deliberately placed **off the board's centre line**.
This gives the BGA enough free space on both sides for a through-hole fanout, so the board **does
not need HDI vias**. It breaks the symmetry of the earlier placement, but the H743 is the only part
moved off-axis. X1, X2 and the mounting holes stay on their DS-012 positions.

### BGA fanout — decided: fine-pitch through-hole dogbone

The STM32H743IIK6 is a **UFBGA-201 at 0.65 mm pitch** in a 10 × 10 mm body, mounted on the
**bottom side**. Two options were on the table: pad depopulation with through-hole fanout, or full
HDI. **Neither was needed.** The fanout is **complete**:

- **Every one of the 201 balls** is fanned out with a dogbone via. Each via sits on the diagonal
  between four balls, 0.46 mm from its own ball.
- **BGA vias are 0.15 mm drill / 0.32 mm diameter**, plated-through Top to Bottom, **filled and
  capped**. There are no microvias, no blind or buried vias, and no via-in-pad on the balls.
- **All other signal vias are 0.2 / 0.42 mm.**
- The fanout escapes on the standard **12-layer, 1.6 mm** stack-up. No balls are depopulated, so
  every peripheral stays usable.

**Routing after the fanout.** Power and the RMII Ethernet group are routed separately, so neither
disturbs the other digital signals. Signal routing from the BGA out to its peripherals has started.

The trade-off that remains is the one standard through-hole vias always carry: via stubs stay in the
stack-up, and the 0.15 mm drill through 1.6 mm is close to a 10.7 : 1 aspect ratio. Both have to be
confirmed against the chosen fab's capability before ordering.

**Open point — microSD shield tab over the fanout.** The H743 is on the bottom and the microSD socket
(CN1) is on the top, directly above it. One of the socket's ground shield tabs (pad 11,
0.7 × 3.33 mm) lands on **10 of the BGA fanout vias**. All 10 are `GND`, the same net as the tab, so
there is no short and no clearance error. The concern is assembly: those vias must be reliably filled
and capped, or solder will wick into them during reflow and weaken the socket's mechanical anchor.

![microSD shield tab over the BGA fanout — outer box: BGA via field, inner box: shield tab](docs/images/bga-fanout-sd-shield.png)

![Naspier Core dimensions](docs/images/dimensions.png)

### Core module renders

| Top | Bottom |
|---|---|
| ![Top render](docs/images/rendered-top.png) | ![Bottom render](docs/images/rendered-bottom.png) |

---

## Repository contents

```
docs/
├── pinout.md                        X1 / X2 / X3 pin tables, generated from the schematic netlist
├── schematic/
│   └── Naspier-Core-Schematic.pdf   Schematic: core module, IMU V2 board, PAB carrier, flex cable
└── images/
    ├── rendered-system-top.png      3D render of all three boards, top
    ├── rendered-system-bottom.png   3D render of all three boards, bottom
    ├── dimensions.png               Core board outline, mounting-hole pattern and dimensions
    ├── rendered-top.png             3D render of the core module, top
    ├── rendered-bottom.png          3D render of the core module, bottom (H743, X1, X2)
    └── bga-fanout-sd-shield.png     microSD shield tab over the BGA fanout
```

The schematic PDF (17 pages) covers, in order:

| Page | Sheet |
|---|---|
| 1 | System block diagram (**preliminary**, still being updated) |
| 2–6 | **Core module:** overview, H743 base (MCU, decoupling, crystals, LEDs, RTC, debug test points), power stage, on-board elements (IMU, baro, FRAM, EEPROM, microSD, I2C pull-ups), connectors X1 / X2 / X3 |
| 7 | **IMU V2 board:** IMU2, IMU3, baro 2, magnetometer, heater, EEPROM, X3 mate |
| 8–14 | **PAB carrier:** overview, X1 / X2 receptacles and mounting, power, IO MCU, elements (Ethernet PHY, gates, EEPROM), two connector sheets |
| 15 | **Flex cable:** X3 to the IMU V2 board |
| 16–17 | System explanation and power-bus index |

> **Editable KiCad source files are not published in this repository** — see
> [what is published here, and what is not](#what-is-published-here-and-what-is-not).

## Status and roadmap

| | |
|---|---|
| Revision | **REV0 VER0** — board ID `NASPIER_CORE_REV0_VER0` |
| Design started | 2026-08-08 |
| Last updated | 2026-10-18 |
| Designer | Ahmet Vapurcu (**naspaw**) |
| Schematic | All three boards drafted and split by board (CORE / SENSOR / CARRIER) plus the flex cable; block diagram preliminary; full review in progress |
| PCB layout — core | **In progress**: 12 layers; BGA fanout complete; power and RMII routed apart from digital signals; routing from the BGA to its peripherals started |
| PCB layout — IMU V2, carrier | Placement started |
| Mechanical | Core mounting holes set (2.3 / 4.0 mm, 31.4 × 24.4 mm); stack clearance to the IMU board, mechanical drawing and IMU vibration isolation still open |
| Fabricated | No |
| Tested | No |
| Firmware | None — no PX4/ArduPilot board target is published for this design |

### Upcoming work

What is queued for this design, roughly in order. This list is the honest state of the project.
The public tracker is the [release checklist](https://github.com/naspaw/NASPIER_CORE/issues/2).

| # | Task | State | What it covers |
|---|---|---|---|
| 1 | **Schematic review** | 🔄 In progress | All three boards: missing and optional parts, wrong labels, ERC to 0 errors |
| 2 | **Sensor, bus and power-bus assignment review** | Open | Every sensor against its bus, chip select, data-ready and switched rail |
| 3 | **Components and voltage buses review** | Open | Part values, ratings and packages; logic levels on I2C / SPI / UART / CAN; rails, loads, sequencing |
| 4 | **Fanout selection** | ✅ Done | Through-hole dogbone, 0.15 / 0.32 mm vias, all 201 balls, 12-layer 1.6 mm |
| 5 | **X1 / X2 footprints** | ✅ Done | Official Pixhawk PAB footprints on core and carrier |
| 6 | **Component placement review** | 🔄 In progress | Core against DS-012 positions; carrier connector positions and orientations |
| 7 | **microSD shield tab over fanout** | Open | Confirm filled + capped vias with the fab, or move the socket off the 10 GND vias |
| 8 | **IMU board assembly design** | Open | How the core and the IMU V2 board stack and mate, flex cable length and orientation |
| 9 | **Stack-up-driven routing** | 🔄 In progress | Core: BGA-to-peripheral signal routing; RMII on its 50 Ω single-ended layer |
| 10 | **Carrier PCB layout** | 🔄 Started | Placement, then routing |
| 11 | **System block diagram** | Open | First version published; still needs updates |
| 12 | **Signal and mechanical drawings** | Open | Core, IMU board and carrier signal drawings; flex cable drawing; assembly drawing |
| 13 | **Vibration isolation mechanics** | Open | Damped mount for the IMU board, with flex-cable slack |
| 14 | **BOM availability check** | ⚠️ Open — issues found | See [supply status](#supply-status) |
| 15 | **Publish layer images + BOM** | Open | Once layout is complete; see the release policy above |

### Supply status

The market can change in the time it takes to lay out a board, and this design shows it. When the project
started on 2026-08-08, both of the parts below were readily available. **As of 2026-09-19, both are close
to out of stock** at DigiKey and at other distributors:

| Part | Role | At project start | Now |
|---|---|---|---|
| **STM32H743IIK6** | MCU | In stock | ⚠️ Close to out of stock |
| **BMI088** | IMU3, on the IMU board | In stock | ⚠️ Close to out of stock |

This also affects the [H743 vs H753](#why-h743-and-not-h753) reasoning. Choosing the H743 removed the
dependency on H753 stock, but it does not protect against the H743 itself running short. Before the
design is frozen, each of these parts needs one of two outcomes: a confirmed source, or a
drop-in / footprint-compatible alternative.

This README will grow as the design does. Changes are listed in the [CHANGELOG](CHANGELOG.md).

## Feedback

**Design review is the point of publishing this.** V0 has not been fabricated or tested, so a second
pair of eyes on the schematic is worth more to this project than anything else. If you spot a
mistake, say so — bluntly is fine.

Particularly useful:

- **Schematic errors** — wrong part, wrong value, missing pull-up or decoupling, a net that does not
  go where the name says it does
- **v6X / PAB compliance** — anything on X1 or X2 that does not match
  [DS-012](https://github.com/pixhawk/Pixhawk-Standards/blob/master/DS-012%20Pixhawk%20Autopilot%20v6X%20Standard.pdf) /
  [DS-010](https://github.com/pixhawk/Pixhawk-Standards/blob/master/DS-010%20Pixhawk%20Autopilot%20Bus%20Standard.pdf),
  or a carrier / module this design would not actually mate with
- **Carrier board** — power mux, protection switches, IO MCU, connector choices
- **BGA fanout**: experience with 0.15 mm through vias on a 1.6 mm, 12-layer board at 0.65 mm pitch.
  Which fabs build it reliably, and what they charge for filled and capped vias
- **Power tree** — rail sequencing, regulator choice, current headroom, monitoring divider values
- **Layout and mechanics** — placement against the DS-012 positions, mounting hole pattern, keep-out
  and stack clearance to the IMU board
- **Sourcing** — a part here that is end-of-life, hard to get, or has a better-stocked equivalent

Two things that are settled and not up for discussion: the choice of **H743 over H753**
(see [above](#why-h743-and-not-h753)), and the fact that **source and production files are not
published**. Everything else is open.

**Contact:** naspawpc@gmail.com

## License

Copyright © 2026 **Ahmet Vapurcu** (**naspaw**). When attributing this design, use that name and a
link back to this repository.

The Naspier Core design documentation in this repository is released under
**[Creative Commons Attribution-ShareAlike 3.0 Unported (CC BY-SA 3.0)](LICENSE)** — the same
license the Pixhawk project uses for its hardware reference designs.

You may use, share, modify and build on **the documents published here**, including commercially,
provided that you **give attribution** and **share any derivative under the same license**.

Note the scope: the license covers what is in this repository. Source and production files are not
published (see [above](#what-is-published-here-and-what-is-not)), so this is open documentation
rather than open-source hardware in the OSHWA sense.

Trademarks, part names and the Pixhawk standards referenced here belong to their respective owners.

## Acknowledgements

Built against the [Pixhawk open standards](https://pixhawk.org/standards/) published by the Pixhawk
Special Interest Group and coordinated by the Dronecode Foundation.
