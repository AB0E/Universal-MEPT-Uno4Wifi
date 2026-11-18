# W3PM Universal MEPT Beacon Firmware & Hardware Guide
### Callsign: AB0E | Grid Square: EM28 | Arduino UNO R4 WiFi & Nextion HMI
### GitHub Repository: https://github.com/AB0E/Universal-MEPT-Uno4Wifi

---

## 1. System Architecture

This project adapts the classic W3PM Ultimate Multi-mode MEPT (Manned Experimental Propagation Transmitter) firmware to modern hardware (replacing all legacy Arduino Nano 33 IoT references):
* **Microcontroller**: Arduino UNO R4 WiFi (Renesas RA4M1 32-bit ARM Cortex-M4 @ 48 MHz)
* **Frequency Synthesizer**: Silicon Labs Si5351A I2C Clock Generator breakout
* **HMI Display**: Nextion NX3224T024 / NX3224T028 UART resistive touch LCD (320x240)
* **Real-Time Clock**: DS3231 High-Precision TCXO RTC on I2C (address `0x68`)
* **Relay Module**: ELEGOO 8-Channel DC 5V Relay Module with Optocoupler (Active-LOW)
* **RF Filtering**: 3-Band LC Band-Pass Filter (BPF) Bank for 40m, 20m, and 10m (Through-Hole Perfboard)
* **Supported Modes**: WSPR-2, FT8, FST4W-120, CW, FSKCW, QRSS, DFCW, Hellschreiber

---

## 2. Arduino UNO R4 WiFi Hardware Pinout & Virtual Touch Mapping

All legacy physical pushbuttons (`PB1`–`PB4` and `MENU`) have been eliminated and replaced by virtual touch codes (`VBTN_PB1` = 101 through `VBTN_MENU` = 105) sent over `Serial1` from the Nextion touch screen.

| Arduino UNO R4 WiFi Pin | Connected Peripheral | Function / Logic Level |
| :--- | :--- | :--- |
| **D0 (RX0)** | Nextion Display TX (Blue wire) | Hardware `Serial1` RX @ 9600 baud |
| **D1 (TX0)** | Nextion Display RX (Yellow wire) | Hardware `Serial1` TX @ 9600 baud |
| **D2** | DS3231 RTC `SQW` Pin | 1PPS Interrupt Input (`FALLING` edge) for millisecond calibration |
| **D3** | External CW Keying Socket | `INPUT_PULLUP` (LOW = internal Si5351 CW; HIGH = key external rig) |
| **D4** | **ELEGOO Relay IN1 (K1)** | **SWR Bridge Relay** (Active-LOW: LOW only during `ANT TUNE`) |
| **D5** | **ELEGOO Relay IN2 & IN3 (K2/K3)** | **40m Bandpass Filter Relay** (Active-LOW: LOW for <= 7.5 MHz) |
| **D6** | **ELEGOO Relay IN4 & IN5 (K4/K5)** | **20m Bandpass Filter Relay** (Active-LOW: LOW for 7.5 - 15.0 MHz) |
| **D7** | **ELEGOO Relay IN6 & IN7 (K6/K7)** | **10m Bandpass Filter Relay** (Active-LOW: LOW for > 15.0 MHz) |
| **D8** | Optional GPS Module `TX` | `gpsSerial` RX @ 9600 baud |
| **D9** | Optional GPS Module `RX` | `gpsSerial` TX @ 9600 baud |
| **D13** | Onboard LED (`LED_BUILTIN`) | Visual CW / FSK keying indicator |
| **A0 (D14)** | Transmit LED / PTT Output | `XmitLED` (HIGH during transmit) |
| **A1 (D15)** | Minute LED | `MinLED` indicator |
| **A2 (D16)** | Second LED | `SecLED` indicator |
| **A3 (D17)** | SWR Bridge Detector Input | Reads analog voltage from 1N34A/1N5711 detector during `ANT TUNE` |
| **A4 (D18 / SDA)** | Si5351A & DS3231 `SDA` | I2C Data Line |
| **A5 (D19 / SCL)** | Si5351A & DS3231 `SCL` | I2C Clock Line |

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
