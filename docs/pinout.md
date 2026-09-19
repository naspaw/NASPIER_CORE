# Naspier Core — Connector pinouts (REV0 VER0)

Three connectors leave the module:

| Ref | Connector | Part | Faces |
|---|---|---|---|
| **X1** | 100-pin, 0.4 mm | Hirose DF40C-100DP-0.4V(58) | v6X carrier board |
| **X2** | 50-pin, 0.4 mm | Hirose DF40C-50DP-0.4V(58) | v6X carrier board |
| **X3** | 34-pin FFC, 0.4 mm | Hirose BM20B-0.8-34DP-0.4V-51 | Naspier V2 IMU board |

> These tables are generated from the REV0 VER0 schematic netlist, not transcribed by hand — they are what
> the schematic actually draws. `NC` means the pin has no net on this module. Signal names are the
> net names used in the schematic; see [the schematic PDF](schematic/Naspier-Core-Schematic.pdf).
>
> **REV0 VER0 is unverified.** Do not use this as a mating reference for a real carrier without checking it
> yourself against
> [DS-012 — Pixhawk Autopilot v6X Standard](https://github.com/pixhawk/Pixhawk-Standards/blob/master/DS-012%20Pixhawk%20Autopilot%20v6X%20Standard.pdf),
> which is the document X1 and X2 are built to.

---

## X1 — 100-pin (CN2)

Follows the Pixhawk v6X 100-pin pinout: 5 V system power, FMU PWM channels 1–8, I2C1–I2C3,
CAN1/CAN2, the full UART set, USB, SWD, ADC inputs and the power-module select lines. Pins 101–104
are the connector hold-downs, tied to GND.

| Pin | Signal | Pin | Signal |
|---:|---|---:|---|
| 1 | `FMU_CH8` | 2 | `GND` |
| 3 | `FMU_CH7` | 4 | `BUZZER` |
| 5 | `FMU_CH6` | 6 | `GND` |
| 7 | `FMU_CH5` | 8 | `I2C3_SDA_BASE_MS5611_BARBED_EXTERNAL1` |
| 9 | `GND` | 10 | `I2C3_SCL_BASE_MS5611_BARBED_EXTERNAL1` |
| 11 | `FMU_CH4` | 12 | `I2C2_SDA_BASE_GPS2_MAG_LED_PM2` |
| 13 | `FMU_CH3` | 14 | `I2C2_SCL_BASE_GPS2_MAG_LED_PM2` |
| 15 | `FMU_CH2` | 16 | `I2C1_SDA_BASE_GPS1_MAG_LED_PM1` |
| 17 | `FMU_CH1` | 18 | `I2C1_SCL_BASE_GPS1_MAG_LED_PM1` |
| 19 | `GND` | 20 | `GND` |
| 21 | `FMU_SAFETY_SWITCH_IN` | 22 | `UART7_RTS_TELEM1` |
| 23 | `FMU_nSAFETY_SWITCH_LED_OUT` | 24 | `UART7_CTS_TELEM1` |
| 25 | `NC_HW_VER_REV_DRIVE` | 26 | `GND` |
| 27 | `NC_HW_VER_SENSE` | 28 | `UART8_TX_GPS2` |
| 29 | `FMU_RTC_3V3` | 30 | `UART8_RX_GPS2` |
| 31 | `GND` | 32 | `GND` |
| 33 | `VDD_3V3_SPEKTRUM_POWER_EN` | 34 | `USART1_RX_GPS1` |
| 35 | `VDD_5V_PERIPH_nEN` | 36 | `USART1_TX_GPS1` |
| 37 | `VDD_5V_PERIPH_nOC` | 38 | `GND` |
| 39 | `FMU_PPM_INPUT` | 40 | `USART2_TX_TELEM3` |
| 41 | `GND` | 42 | `USART2_RX_TELEM3` |
| 43 | `GND` | 44 | `GND` |
| 45 | `GND` | 46 | `USART2_RTS_TELEM3` |
| 47 | `GND` | 48 | `USART2_CTS_TELEM3` |
| 49 | `VDD_5V_SYS` | 50 | `GND` |
| 51 | `VDD_5V_SYS` | 52 | `UART5_TX_TELEM2` |
| 53 | `VDD_5V_SYS` | 54 | `UART5_RX_TELEM2` |
| 55 | `VDD_5V_SYS` | 56 | `GND` |
| 57 | `CAN2_TX` | 58 | `UART5_RTS_TELEM2` |
| 59 | `CAN2_RX` | 60 | `UART5_CTS_TELEM2` |
| 61 | `GND` | 62 | `GND` |
| 63 | `CAN1_TX` | 64 | `UART7_TX_TELEM1` |
| 65 | `CAN1_RX` | 66 | `UART7_RX_TELEM1` |
| 67 | `GND` | 68 | `GND` |
| 69 | `USART3_TX_DEBUG` | 70 | `USART6_RX_FROM_IO__RC_INPUT` |
| 71 | `USART3_RX_DEBUG` | 72 | `USART6_TX_TO_IO__NC` |
| 73 | `GND` | 74 | `GND` |
| 75 | `FMU_SWDIO` | 76 | `USB_D_P` |
| 77 | `FMU_SWCLK` | 78 | `USB_D_N` |
| 79 | `GND` | 80 | `VBUS_SENSE` |
| 81 | `VDD_5V_HIPOWER_nEN` | 82 | `GND` |
| 83 | `VDD_5V_HIPOWER_nOC` | 84 | `VDD_3V3_AUX` |
| 85 | `nARMED` | 86 | `VDD_3V3_AUX` |
| 87 | `FMU_nRST` | 88 | `GND` |
| 89 | `nPOWER_IN_A` | 90 | `ADC1_6V6` |
| 91 | `nPOWER_IN_B` | 92 | `ADC1_3V3` |
| 93 | `nPOWER_IN_C` | 94 | `GND` |
| 95 | `GND` | 96 | `UART4_RX` |
| 97 | `FMU_CAP1` | 98 | `UART4_TX` |
| 99 | `GND` | 100 | `GND` |
| 101 | `GND` | 102 | `GND` |
| 103 | `GND` | 104 | `GND` |

---

## X2 — 50-pin (CN4)

Carries 100BASE-T RMII Ethernet, the external SPI6 bus, `SPIX_SYNC` and `PG6`.

**Fifteen pins are left unconnected on purpose.** The schematic annotates each of them with the
function the later **v6X-RT** standard update assigns there, under the note *"Latest Standard
Updates, Based on v6X-RT — Those are NOT COVERED ON v6X."* Those names appear below as *reserved*:
they document what the pin would become, they are **not** signals this board drives. On a plain v6X
carrier the positions are undefined anyway; on a v6X-RT carrier they simply go unused.

This means Naspier Core REV0 VER0 does **not** bring out PWM channels 9–12 or CAN3 — those are v6X-RT
additions, and wiring them is a candidate for a later revision.

| Pin | Signal | Pin | Signal |
|---:|---|---:|---|
| 1 | `GND` | 2 | `ETH_MDIO` |
| 3 | `ETH_REF_CLK` | 4 | `ETH_MDC` |
| 5 | `GND` | 6 | `ETH_POWER_EN` |
| 7 | `ETH_CRS_DV` | 8 | `GND` |
| 9 | `GND` | 10 | *NC* — reserved: `ETH_RX_ER` |
| 11 | `ETH_RXD0` | 12 | *NC* — reserved: `ETH_PHY_nINT` |
| 13 | `GND` | 14 | `GND` |
| 15 | `ETH_RXD1` | 16 | *NC* — reserved: `FMU_CH9` |
| 17 | `GND` | 18 | *NC* — reserved: `FMU_CH10` |
| 19 | `ETH_TXD0` | 20 | *NC* — reserved: `FMU_CH11` |
| 21 | `GND` | 22 | *NC* — reserved: `FMU_CH12` |
| 23 | `ETH_TXD1` | 24 | `GND` |
| 25 | `GND` | 26 | *NC* — reserved: `SPARE09` |
| 27 | `ETH_TX_EN` | 28 | *NC* — reserved: `SPARE10` |
| 29 | `GND` | 30 | *NC* — reserved: `SPARE11` |
| 31 | `SPI6_MISO_EXTERNAL1` | 32 | *NC* — reserved: `SPARE12` |
| 33 | `SPI6_MOSI_EXTERNAL1` | 34 | `GND` |
| 35 | `SPI6_SCK_EXTERNAL1` | 36 | *NC* — reserved: `SPARE14` |
| 37 | `GND` | 38 | *NC* — reserved: `SPARE15` |
| 39 | `SPI6_nRESET_EXTERNAL1` | 40 | *NC* — reserved: `CAN3_TX` |
| 41 | `SPI6_nCS1_EXTERNAL1` | 42 | *NC* — reserved: `CAN3_RX` |
| 43 | `SPI6_nCS2_EXTERNAL1` | 44 | `PG6` |
| 45 | `SPI6_DRDY2_EXTERNAL1` | 46 | `GND` |
| 47 | `SPI6_DRDY1_EXTERNAL1` | 48 | `GND` |
| 49 | `SPIX_SYNC` | 50 | *NC* — reserved: `PH11` |
| 51 | `GND` | 52 | `GND` |
| 53 | `GND` | 54 | `GND` |

---

## X3 — 34-pin FFC (CN5)

**This connector is not a Pixhawk standard connector.** It is a private link between Naspier Core
and the Naspier V2 IMU board, and it never faces the carrier. It carries:

- **SPI2** to the LSM6DSV32X (IMU2), one chip select and a data-ready line
- **SPI3** to the BMI088 accelerometer + gyroscope (IMU3), two chip selects and a data-ready line
- **I2C4** plus the BMP581 barometer interrupt and RM3100 magnetometer data-ready lines
- **Power:** `VDD_SENSOR_BUS_2`, `VDD_SENSOR_BUS_3`, `VDD_SENSOR_BUS_4` (three of the four switched
  sensor rails) and `VDD_5V_SYS`
- **`HEATER`** — sensor heater drive
- Pins 35–38 are the FFC shield tabs, tied to GND

Two pins are flagged in the schematic itself as departures even from the sensor-connector
convention, marked below: **pin 11** (`I2C4_INT_BARO1_BMP581`) and **pin 13**
(`I2C4_DRDY_MAG_RM3100`).

| Pin | Signal | Pin | Signal |
|---:|---|---:|---|
| 1 | `GND` | 2 | `SPI3_nCS1_BMI088_ACCEL` |
| 3 | `SPI3_DRDY2_BMI088_INT3_GYRO` | 4 | `SPI3_nCS2_BMI088_GYRO` |
| 5 | `I2C4_SDA_FMU` | 6 | `VDD_SENSOR_BUS_3` |
| 7 | `I2C4_SCL_FMU` | 8 | `GND` |
| 9 | `GND` | 10 | `SPI3_SCK_IMU3_BMI088` |
| 11 | `I2C4_INT_BARO1_BMP581` ⚠️ | 12 | `SPI3_MISO_IMU3_BMI088` |
| 13 | `I2C4_DRDY_MAG_RM3100` ⚠️ | 14 | `SPI3_MOSI_IMU3_BMI088` |
| 15 | `GND` | 16 | `VDD_SENSOR_BUS_4` |
| 17 | `GND` | 18 | `GND` |
| 19 | `GND` | 20 | `SPI2_SCK_IMU2_LSM6DSV` |
| 21 | `GND` | 22 | `SPI2_MISO_IMU2_LSM6DSV` |
| 23 | `GND` | 24 | `SPI2_MOSI_IMU2_LSM6DSV` |
| 25 | `GND` | 26 | `GND` |
| 27 | `GND` | 28 | `VDD_5V_SYS` |
| 29 | `GND` | 30 | `HEATER` |
| 31 | `SPI2_DRDY1_LSM6DSV_INT1` | 32 | `VDD_SENSOR_BUS_2` |
| 33 | `SPI2_nCS1_LSM6DSV` | 34 | `GND` |
| 35 | `GND` | 36 | `GND` |
| 37 | `GND` | 38 | `GND` |
