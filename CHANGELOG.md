# Changelog

All notable changes to the Naspier Core hardware design are recorded here.

## [REV0 VER0] — 2026-10-15 · schematic restructure and carrier board

Still REV0 VER0, still not fabricated or tested. **The schematic is now split by board, and the
PAB carrier board schematic is drafted.** The schematic PDF is re-plotted with all three boards
(14 pages). README and `pinout.md` will follow once the schematic review is finished
([release checklist](https://github.com/naspaw/NASPIER_CORE/issues/2)).

### Schematic structure

- Sheets are grouped by board: **CORE** (H743, power, onboard elements, connectors), **SENSOR**
  (Naspier V2 IMU board) and **CARRIER** (PAB carrier board). Sheet and symbol identifiers are kept,
  so the PCB link is unchanged.

### Core module

- **X1 pin 87 now reaches the H743 `NRST`.** The reset net was split by a label name mismatch
  (`FMU_NRST` / `FMU_nRST`).

### Carrier board (new)

- Power tree with LTC4417 power mux, peripheral and high-power current limiters, IO MCU
  (STM32F103), B2B receptacles for X1 / X2, peripheral and high-power connector sheets.
- Added from the PAB-BASE RC05 reference, redrawn by hand:
  - FMU debug and IO debug connectors (SM10B)
  - RC IN header with an S.BUS buffer, and an FMU reset-request gate
  - AD&IO port with ADC input conditioning on `ADC1_3V3` / `ADC1_6V6`
  - External SPI6 port (BM11B)
- Fixed: TELEM2 / TELEM3 and GPS2 / UART4 + I2C3 were shorted, the safety switch LED was not
  connected, and the carrier EEPROM was not on I2C3.

### Open items

- [ ] Schematic ERC to 0 errors
- [ ] Schematic review, including the carrier board
- [ ] Components and voltage buses review: part ratings, logic levels, power rails
- [x] Schematic PDF re-plotted: core, IMU and carrier boards
- [ ] README and `pinout.md` updated to the new structure
- [ ] Carrier board PCB (not started)

## [REV0 VER0] — 2026-09-30 · documentation errata

A review found mistakes in the published README and `pinout.md`. The corrections are listed here.
The README, `pinout.md` and schematic PDF themselves will be updated together with the carrier
board documentation.

### Corrections

- **`nPOWER_IN_A/B/C` (X1 pins 89, 91, 93) are inputs to the FMU, not power-module select lines.**
  They carry the active-low "input valid" status from the carrier's power mux (LTC4417
  `VALID1..3`). The power mux chooses the source itself.
- **Direction of the carrier power control pins, seen from the FMU:**
  - `VDD_5V_PERIPH_nEN`, `VDD_5V_HIPOWER_nEN` and `VDD_3V3_SPEKTRUM_POWER_EN` are outputs.
  - `VDD_5V_PERIPH_nOC` and `VDD_5V_HIPOWER_nOC` are inputs (fault / over-current flags).
- **`ADC1_3V3` / `ADC1_6V6` (X1 pins 92, 90) are general-purpose analog inputs**, 3.3 V and 6.6 V
  full-scale. They are not power-module sensing. Battery voltage and current come from digital power
  modules on I2C1 / I2C2.
- **RTS/CTS flow control** is on TELEM1 (UART7), TELEM2 (UART5) and TELEM3 (USART2) only. The other
  UARTs are 2-wire.
- **USART6 is the 2-wire link to the carrier's IO MCU** (STM32F103), which decodes the RC receiver
  signal. The README listed it as "RC input", which is only true on carriers without an IO MCU.
- **The H743 pin map follows the as-built v6X map** used by PX4 `fmu-v6x` and ArduPilot `Pixhawk6X`,
  not the pin table in DS-012 rev 0.4.0. The X1 / X2 connector pinouts are the same in both.
- **Sensor rails:** the BMP581 is on `VDD_SENSOR_BUS_4` (listed as `VDD_SENSOR_BUS_2`), and the
  core-module AT24C64D is on `VDD_SENSOR_BUS_1` (listed as `VDD_3V3_AUX`).
- **RTC charge path** is D1 and R3 (listed as D1 and R2).
- **`pinout.md`, X2 pin 48** is `NFC_GPIO` (listed as `GND`).
- **`pinout.md`, X2 pins 14, 24 and 34** are GND, following the v6X-RT layout. DS-010 lists them as
  `SPARE03`, `SPARE08` and `SPARE13`.

## [REV0 VER0] — 2026-09-19 · layout update

Still REV0 VER0, still not fabricated or tested. **The PCB layout moved forward and the BGA fanout
question is closed.**

### Schematic

- Added the **Naspier V2 IMU board** base schematic (sheet 6):
  - LSM6DSV32X on SPI2
  - BMI088 on SPI3
  - BMP581 and RM3100 on I2C4
  - AT24C64D EEPROM at 0x50 on I2C4
  - AO3400A heater driver with 51 Ω / 1 W loads
  - X3 mate
- Added the **system block diagram** as a first version. It still needs updates.
- Debug: removed the 8-pin GH1.25 debug connector. Debug and programming now go through test points:
  - TP6 `FMU_nRST`, TP7 `FMU_SWDIO`, TP8 `VDD_REG_FMU_3V3`, TP9 `FMU_SWCLK`, TP10 `GND`
  - TP15 / TP16 `USART3_TX_DEBUG` / `USART3_RX_DEBUG`, TP17 `TRACECLK`
  - SPI4 expansion moved to TP11–TP14
- FRAM `WP` / `HOLD` now go to test points TP2 / TP1 and rely on the part's internal pull-ups.
- IMU board power: the IMU-board EEPROM is now on `VDD_SENSOR_BUS_4` (X3 does not carry
  `VDD_3V3_AUX`). BMP581 shares `VDD_SENSOR_BUS_2` with the LSM6DSV32X.
- X3 pin 31 is now `SPI2_DRDY1_LSM6DSV_INT1` (was an unused reserved pin).

### PCB

- **Component placement revised** for signal routing. The STM32H743 is deliberately placed off the
  board's centre line to make room for a through-hole fanout without HDI vias.
- **BGA fanout decided and complete:**
  - All 201 balls escape with through-hole dogbone vias of **0.15 / 0.32 mm**, filled and capped.
  - It fits the 12-layer, 1.6 mm stack-up, with no HDI and no pad depopulation.
- All other signal vias are **0.2 / 0.42 mm**.
- **Routing started** on the 12-layer stack-up. Power and the RMII group are routed apart from the
  other digital signals, and signal routing from the BGA to its peripherals is under way.
- Mounting holes are **2.3 mm drill / 4.0 mm pad** on a **31.4 × 24.4 mm** pattern. The earlier
  README value of 2.2 mm was wrong.
- **Known open point:** the microSD socket's ground shield tab (CN1 pad 11) sits over 10 GND fanout
  vias. There is no short, because it is the same net, but those vias must be filled and capped to
  avoid solder wicking.

### Documentation

- Updated the renders, dimensions drawing and schematic PDF. Added a close-up of the fanout / microSD
  overlap.
- `pinout.md`: corrected the connector designators to X1 = CN2, X2 = CN4, X3 = CN5, and IMU2 to
  LSM6DSV32X.
- README: removed the power tree diagram.
- README: corrected the H753 wording — DS-012 calls for an STM32H7; the H753 is what shipping v6X
  boards use. I2C4 is no longer listed as a carrier interface (it only goes to X3).

### Open items

- [ ] Schematic review, including the IMU board
- [x] Fanout selection
- [ ] Component placement review against DS-012 positions (in progress)
- [ ] Mounting hole keep-outs and stack clearance
- [ ] microSD shield tab over fanout vias: confirm fill and cap with the fab, or move the socket
- [ ] IMU board assembly design
- [ ] Stack-up-driven routing (in progress)
- [ ] System block diagram update
- [ ] BOM availability check

## [REV0 VER0] — 2026-09-13

Initial design revision, board ID `NASPIER_CORE_REV0_VER0`, by Ahmet Vapurcu (VAP-SAN); design
started 2026-08-08. **Schematic drafted; PCB layout and mechanical design not finished; not
fabricated, not tested.**

### Core

- STM32H743IIK6 (UFBGA-201, 10 × 10 mm, 0.65 mm pitch), 480 MHz, 2 MB flash / 1 MB RAM
- H743 chosen over the H753 used in shipping v6X boards: no crypto/hash accelerator needed for this target
- 16 MHz main oscillator, 32.768 kHz RTC oscillator, coin-cell RTC backup
- Discrete R/G/B status LEDs

### Interfaces

- Pixhawk v6X board-to-board interface: DF40C-100DP-0.4V (X1) + DF40C-50DP-0.4V (X2)
- X1: 5 V system power, PWM `FMU_CH1`–`CH8`, I2C1–I2C3, CAN1 + CAN2, UART1–UART8, USB, SWD,
  ADC sense, power-module select A/B/C, safety switch, buzzer
- X2: 100BASE-T RMII Ethernet, external SPI6, `SPIX_SYNC`, `PG6`
- X2: fifteen pins left as no-connects, annotated with the functions the later v6X-RT update
  assigns there (`FMU_CH9`–`CH12`, `CAN3`, `SPARE09`–`SPARE12`, `SPARE14`, `SPARE15`, `ETH_RX_ER`, `ETH_PHY_nINT`,
  `PH11`). **PWM 9–12 and CAN3 are therefore not available on this revision.**
- X3: 34-pin FFC (BM20B-0.8-34DP-0.4V-51) to the Naspier V2 IMU board — custom pinout, does not
  face the carrier
- Debug test-point group: TP10 (`VDD_REG_FMU_3V3`), TP13 (`FMU_SWDIO`), TP14 (`FMU_SWCLK`),
  TP16 (`FMU_nRST`), TP17 (`GND`), plus TP6–TP9 on the unused SPI4 expansion bus; an 8-pin GH1.25
  connector (CN4) is drawn alongside, annotated as serving that group
- RTC backup is an MS621FE-FL11E rechargeable cell; its charge path (D1, R2) is flagged do-not-place

### Sensors and storage

- On-board: ICM-45686 IMU (SPI1), MS5611 barometer (I2C3), FM25V02A FRAM (SPI5),
  AT24C64D EEPROM at 0x50 (I2C3), microSD over SDMMC2 4-bit
- Over X3: LSM6DSV (SPI2), BMI088 (SPI3), BMP581 and RM3100 interrupt/data-ready (I2C4),
  sensor heater drive

### Power

- 5 V in → 1 µH / 3.1 A line filter → 2 × MCP1727T-3302E/MF (FMU 3V3 with power-good, AUX 3V3 with
  power-good) + 5 × LDL212PV33R (four switched sensor rails + SD card rail)
- Every sensor rail independently enabled and independently voltage-scaled back to the ADC
- 5 V input monitoring via divider; digital and analog FMU 3V3 separated through NFM18PC104R1C3D
  EMI filters

### PCB

- 12-layer FR4, ≈1.6 mm, blue soldermask, tented vias
- 29.00 × 36.00 mm, four corner mounting holes
- Board outline, X1/X2 positions, mounting holes and keep-outs placed per DS-012
- RMII group in a 50 Ω single-ended impedance-controlled zone
- **BGA fanout strategy undecided** — pad depopulation with through-hole fanout, or full HDI

### Published

- Documentation only: schematic PDF, connector pinouts, dimensions, 3D renders
- PCB layer images and BOM to follow once layout is complete
- Source files and production files are not published

### Open items

- [ ] Schematic review
- [ ] Fanout selection
- [ ] Component placement review against DS-012 positions and mechanical tolerances
- [ ] Mounting hole design
- [ ] IMU board assembly design, including whether it needs its own mounting holes
- [ ] Stack-up-driven routing: RMII impedance layer, digital and power layer separation
- [ ] BOM availability check
