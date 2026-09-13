# Changelog

All notable changes to the Naspier Core hardware design are recorded here.

## [REV0 VER0] — 2026-09-13

Initial design revision, board ID `NASPIER_CORE_REV0_VER0`, by Ahmet Vapurcu (VAP-SAN); design
started 2026-08-08. **Schematic drafted; PCB layout and mechanical design not finished; not
fabricated, not tested.**

### Core

- STM32H743IIK6 (UFBGA-201, 10 × 10 mm, 0.65 mm pitch), 480 MHz, 2 MB flash / 1 MB RAM
- H743 chosen over the DS-012-specified H753: no crypto/hash accelerator needed for this target
- 16 MHz main oscillator, 32.768 kHz RTC oscillator, coin-cell RTC backup
- Discrete R/G/B status LEDs

### Interfaces

- Pixhawk v6X board-to-board interface: DF40C-100DP-0.4V (X1) + DF40C-50DP-0.4V (X2)
- X1: 5 V system power, PWM `FMU_CH1`–`CH8`, I2C1–I2C4, CAN1 + CAN2, UART1–UART8, USB, SWD,
  ADC sense, power-module select A/B/C, safety switch, buzzer
- X2: 100BASE-T RMII Ethernet, external SPI6, `SPIX_SYNC`, `PG6`
- X2: fifteen pins left as no-connects, annotated with the functions the later v6X-RT update
  assigns there (`FMU_CH9`–`CH12`, `CAN3`, `SPARE09`–`SPARE15`, `ETH_RX_ER`, `ETH_PHY_nINT`,
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
