# W3PM Universal MEPT Beacon Firmware & Hardware Guide
### Callsign: AB0E | Grid Square: EM28 | Arduino UNO R4 WiFi & Nextion HMI

---

## 1. System Architecture

This project adapts the classic W3PM Ultimate Multi-mode MEPT (Manned / Manned Experimental Propagation Transmitter) firmware to modern 2026 hardware:
* **Microcontroller**: Arduino UNO R4 WiFi (Renesas RA4M1 32-bit ARM Cortex-M4 @ 48 MHz)
* **Frequency Synthesizer**: Silicon Labs Si5351A I2C Clock Generator breakout
* **HMI Display**: Nextion NX3224T024 2.8" UART resistive touch LCD (320x240)
* **Real-Time Clock**: DS3231 High-Precision TCXO RTC on I2C (address `0x68`)
* **Relay Module**: ELEGOO 8-Channel DC 5V Relay Module with Optocoupler (Active-LOW)
* **Supported Modes**: WSPR-2, FT8, FST4W-120, CW, FSKCW, QRSS, DFCW, Hellschreiber

---

## 2. Hardware Pinout & Wiring Table

| Arduino UNO R4 Pin | Hardware Peripheral | Connection Details / Logic Level |
| :--- | :--- | :--- |
| **D0 (RX1)** | Nextion Display TX (Blue wire) | Hardware `Serial1` RX (3.3V / 5V tolerant) |
| **D1 (TX1)** | Nextion Display RX (Yellow wire) | Hardware `Serial1` TX (Direct connection) |
| **D2** | PushButton 1 / Touch Interrupt | Internal Pullup (Virtual mapping to Nextion PB1) |
| **D3** | PushButton 2 / Touch Interrupt | Internal Pullup (Virtual mapping to Nextion PB2) |
| **D4** | **SWR Bridge Relay (IN1)** | **Active-LOW**: Energized ONLY during ANT TUNE |
| **D5** | **40m Lowpass Filter Relay (IN2/3)** | **Active-LOW**: Energized for <= 7.5 MHz |
| **D6** | **20m Lowpass Filter Relay (IN4/5)** | **Active-LOW**: Energized for 7.5 - 15.0 MHz |
| **D7** | **10m Lowpass Filter Relay (IN6/7)** | **Active-LOW**: Energized for > 15.0 MHz |
| **D8 (RX)** | Optional GPS Module TX | SoftwareSerial / NMEA 9600 baud GPS receiver |
| **D9 (TX)** | Optional GPS Module RX | SoftwareSerial / NMEA 9600 baud GPS receiver |
| **D10** | PushButton 3 | Internal Pullup (Virtual mapping to Nextion PB3) |
| **D11** | PushButton 4 | Internal Pullup (Virtual mapping to Nextion PB4) |
| **A4 / SDA** | Si5351A & DS3231 SDA | I2C Data bus |
| **A5 / SCL** | Si5351A & DS3231 SCL | I2C Clock bus |
| **A0** | SWR Forward Voltage ($V_f$) | Analog input (0 - 5V) from resistive bridge |
| **A1** | SWR Reverse Voltage ($V_r$) | Analog input (0 - 5V) from resistive bridge |

---

## 3. Power Supply Distribution (Dual Rail: 5V & 3.3V)

Using your dual-rail breadboard power supply module:
* **Rail 1 (+5.0V DC)**:
  * Powers **Arduino UNO R4 WiFi** (via 5V pin or USB-C)
  * Powers **Nextion NX3224T024 Display** (+5V red wire, consumes ~150 mA)
  * Powers **ELEGOO 8-Channel Relay Module VCC** (removes current draw from Arduino regulator)
* **Rail 2 (+3.3V DC)**:
  * Powers **Si5351A Synthesizer Module** (VCC 3.3V)
  * Powers **DS3231 RTC Module** (VCC 3.3V)
* **Ground (GND)**:
  * **MANDATORY**: Common ground connected between all boards, power rails, relays, and display.

---

## 4. ELEGOO 8-Channel Active-LOW Relay Operation

The relay module inputs (`IN1` through `IN8`) use optocouplers:
* Setting an Arduino digital pin **LOW (0V)** turns the relay **ON** (contacts closed).
* Setting an Arduino digital pin **HIGH (+5V)** turns the relay **OFF** (contacts open).
* In `w3pm_UltimateMEPT_V2_1.ino`, all relay pins are pre-loaded `HIGH` before `pinMode(OUTPUT)` to prevent contact chattering during power-up.

---

## 5. Nextion Display Setup

1. Copy `nextion_screen_design.md` settings into **Nextion Editor**.
2. Generate Fonts: Font 0 (16px), Font 1 (24px bold), Font 2 (20px bold).
3. Place text components `t0`, `t1`, `t2`, `t3` and buttons `b0` through `b5`.
4. Compile to `.tft` file and flash via FAT32 MicroSD card.
