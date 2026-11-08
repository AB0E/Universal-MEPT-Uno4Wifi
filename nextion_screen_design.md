# Nextion 2.8" Touch Screen Hierarchical Menu & UI Architecture
## Station AB0E | Grid EM28 | Rig: MEPT Beacon (UNO R4 WiFi & NX3224T024)
### GitHub Repository: https://github.com/AB0E/W3PM-Nano-33-IOT-Universal-MEPT

---

## 1. Modern Touch Menu Hierarchy (No Physical Pushbuttons)

In place of legacy physical buttons (`PB1`, `PB2`, `PB3`, `PB4`), the Nextion 2.8" touch screen utilizes a modern, nested menu-to-submenu hierarchy. Tapping any category tile on the **Main Menu** opens a dedicated graphical **Sub-Menu**, complete with live readouts, steppers, and a universal `[ < BACK ]` button.

```
                    +------------------------------------+
                    |        PAGE 0: MAIN DASHBOARD      |
                    |  (Live Telemetry, RF Status, Pwr)  |
                    +-----------------+------------------+
                                      |
                           [ TAP "MAIN MENU" ]
                                      |
                                      v
                    +------------------------------------+
                    |        PAGE 1: MAIN MENU GRID      |
                    |  6 Core Operational Sub-Categories |
                    +-----------------+------------------+
                                      |
         +------------+---------------+------------+------------+------------+
         |            |               |            |            |            |
         v            v               v            v            v            v
     [SUB-MENU 1] [SUB-MENU 2]    [SUB-MENU 3] [SUB-MENU 4] [SUB-MENU 5] [SUB-MENU 6]
      MODULATION   RF BANDS &     ANT TUNE &    TIMING &     STATION &    SPACE WX &
      SELECTION    LPF RELAYS     SWR METER     CALIBRATION  POWER        RELAY TEST
       (Page 2)     (Page 3)       (Page 4)      (Page 5)     (Page 6)     (Page 7)
```

---

## 2. Menu Structure & Sub-Menu Directory

### [Level 0] Page 0: Main Dashboard (`main`)
* **Header**: Callsign `AB0E` | Grid `EM28` | Power `7 dBm` (5 mW) | `[ MENU > ]` button.
* **Live Telemetry (Streamed from Arduino)**:
  * Line 0 (`t0`): Real-time UTC Clock & GPS Lock status (`21:47:18 UTC GPS LOCKED`).
  * Line 1 (`t1`): Active Mode, Band & Dial Frequency (`WSPR-2 20m 14.097100 MHz`).
  * Line 2 (`t2`): Forward Power & Live SWR readout (`Tx: 7dBm  SWR: 1.15 : 1`).
  * Line 3 (`t3`): Active Filter Relay & Countdown (`20m LPF [D6 ON] Next: 00:42`).
* **Visual Progress Bar (`j0`)**: Active sequence countdown (0–100%).
* **Quick Touch Bar**: `[ DOWN - ]`, `[ SELECT ]`, `[ NEXT > ]`, `[ UP + ]`, `[ SAVE ]`, `[ MENU ]`.

---

### [Level 1] Page 1: Main Menu Hub (`menu_hub`)
Presents 6 clean, colorful touch tiles linking directly into dedicated sub-menus:
1. **`[ 1. MODULATION MODES ]`** $
ightarrow$ Jumps to Page 2
2. **`[ 2. BANDS & LPF FILTERS ]`** $
ightarrow$ Jumps to Page 3
3. **`[ 3. ANTENNA TUNE & SWR ]`** $
ightarrow$ Jumps to Page 4
4. **`[ 4. TIMING & FREQ CALIB ]`** $
ightarrow$ Jumps to Page 5
5. **`[ 5. STATION & POWER ]`** $
ightarrow$ Jumps to Page 6
6. **`[ 6. RELAY DIAGNOSTICS ]`** $
ightarrow$ Jumps to Page 7
* Bottom Bar: `[ < RETURN TO DASHBOARD ]` $
ightarrow$ Jumps to Page 0.

---

### [Level 2] Sub-Menu Detail Specifications

#### Sub-Menu 1: Modulation Modes (`sub_mode` - Page 2)
Direct 1-touch mode selection with visual radio buttons:
* `( ) WSPR-2` (162 tones, 2m even UTC minutes) $
ightarrow$ `prstr "M1"`
* `( ) FT8` (79 tones, 15s intervals :00, :15, :30, :45s) $
ightarrow$ `prstr "M2"`
* `( ) FST4W-120` (160 tones, 120s even UTC minutes) $
ightarrow$ `prstr "M3"`
* `( ) CW` (12 WPM Morse ID: `AB0E EM28`) $
ightarrow$ `prstr "M4"`
* `( ) FSKCW` (Continuous carrier with 5 Hz FSK shift) $
ightarrow$ `prstr "M5"`
* `( ) QRSS` (QRSS3 slow Morse, 3s dit / 9s dah) $
ightarrow$ `prstr "M6"`
* `( ) DFCW` (Dual Frequency CW, 3s elements on $f_0$ and $f_0+6	ext{ Hz}$) $
ightarrow$ `prstr "M7"`
* `( ) Hellschreiber` (Feld-Hell 122.5 baud column scanning) $
ightarrow$ `prstr "M8"`
* Controls: `[ APPLY & SAVE ]` (sends `'S'`), `[ < BACK TO MENU ]` (returns to Page 1).

#### Sub-Menu 2: Bands & LPF Filters (`sub_band` - Page 3)
* Direct band selection matrix:
  * **40m** (7.038 / 7.074 MHz) $
ightarrow$ Energizes Relays **K2 & K3 (Pin D5)**
  * **20m** (14.074 / 14.097 MHz) $
ightarrow$ Energizes Relays **K4 & K5 (Pin D6)**
  * **10m** (28.074 / 28.124 MHz) $
ightarrow$ Energizes Relays **K6 & K7 (Pin D7)**
  * Other bands (160m, 80m, 30m, 17m, 15m, 12m, 6m)
* Live Filter Status Card: Displays toroid core specifications (**FT 37-43**) and active relay coil indicators.

#### Sub-Menu 3: Antenna Tune & SWR Meter (`sub_swr` - Page 4)
* Visual analog-style SWR bar gauge with color threshold zones:
  * Green: SWR 1.0 – 1.5 (Safe)
  * Amber: SWR 1.5 – 2.5 (Marginal)
  * Red: SWR > 2.5 (High mismatch)
* Live Voltage Readouts: $V_f$ Forward Voltage (A0) and $V_r$ Reverse Voltage (A1).
* Controls:
  * `[ START TUNE ]`: Pulls Arduino pin **D4 LOW**, energizing Relay **K1** to switch the 50 $\Omega$ bridge in-line and emit a carrier.
  * `[ STOP & BYPASS ]`: De-energizes Relay **K1** (bypassed) and shuts off RF.
  * `[ < BACK TO MENU ]`

#### Sub-Menu 4: Timing & GPS Calibration (`sub_timing` - Page 5)
* **Timing Source Options (WiFi & GPS Only)**:
  * `[ WiFi Internet NTP Sync ]`: Connects via UNO R4 WiFi.
  * `[ GPS NMEA Satellite Sync ]`: Reads hardware serial on pin `D8`.
* **GPS-Disciplined Frequency Calibration**:
  * Emits a 25.000000 MHz reference signal on CLK1.
  * `[ VERIFY GPS LOCK ]`: Checks satellite fix ($>4$ sats) and timing stability.
  * Steppers: `[-10 Hz]`, `[-1 Hz]`, `[+1 Hz]`, `[+10 Hz]`.
  * `[ SAVE CAL TO EEPROM ]`: Saves parts-per-billion offset permanently.

#### Sub-Menu 5: Station Identity & Power (`sub_station` - Page 6)
* Callsign: **`AB0E`**
* Maidenhead Grid: **`EM28`**
* Transmit Power Stepper: **`7 dBm`** (5.0 mW EIRP) with `[-1 dBm]` and `[+1 dBm]` buttons.
* Transmit Interval Selector: `Continuous`, `2 min`, `4 min`, `6 min`, `10 min`, `15 min`, `20 min`, `30 min`.
* `[ SAVE CONFIG ]` $
ightarrow$ Stores values in Arduino EEPROM.

#### Sub-Menu 6: Relay Diagnostics & Space Weather (`sub_diag` - Page 7)
* **Elegoo 8-Channel Relay Diagnostic Board**:
  * Real-time toggles to individually click test each relay coil:
    * `Relay 1 (D4)`: SWR Bridge
    * `Relays 2-3 (D5)`: 40m LPF
    * `Relays 4-5 (D6)`: 20m LPF
    * `Relays 6-7 (D7)`: 10m LPF
* **Solar & Space Weather**: SFI, Sunspot Number (SSN), A-Index, K-Index.

---

## 3. GitHub Repository Synchronization

All firmware source files, Nextion HMI layouts, and wiring schematics align directly with your project repository:
* **Repository**: [`https://github.com/AB0E/W3PM-Nano-33-IOT-Universal-MEPT`](https://github.com/AB0E/W3PM-Nano-33-IOT-Universal-MEPT)
* **Branch Target**: `main`
* **Directory Structure**:
  * `/w3pm_UltimateMEPT_V2_1.ino` (Arduino sketch)
  * `/nextion_screen_design.md` (This document)
  * `/nextion_ui_simulator.html` (Interactive browser simulator)
  * `/w3pm_style_system_wiring_diagram.png` (Publication wiring schematic)
  * `/lpf_40m_toroid_schematic.png` (40m FT 37-43 LPF schematic)
  * `/lpf_20m_toroid_schematic.png` (20m FT 37-43 LPF schematic)
  * `/lpf_10m_toroid_schematic.png` (10m FT 37-43 LPF schematic)
  * `/WSPR_FT8_FST4W_Symbols.txt` (Mode specifications & symbol tables)

---

## 4. WiFi Scanner & On-Screen Touch Keyboard Pages

To eliminate hardcoded credentials in the `.ino` file, two dedicated touch pages handle dynamic network discovery and password input:

### Page: `wifi_scan` (Available Network Selection)
* **Trigger**: Tapping `[ 1. WiFi NTP Sync ]` or `[ SCAN NETWORKS ]`. Sends `'N'` to the Arduino to initiate `WiFi.scanNetworks()`.
* **Components**:
  * `t_hdr` (Text): `"SELECT WIFI NETWORK"`
  * `t_stat` (Text): `"Scanning networks..."` / `"Select an SSID below"`
  * Network List Buttons (`b_net0` through `b_net7`):
    * Populated dynamically by the Arduino with SSID names and RSSI signal strengths (e.g. `"HomeNetwork (-62 dBm)"`).
    * **Touch Press Event**:
      * `b_net0`: `prstr "W0"` (selects 1st scanned network)
      * `b_net1`: `prstr "W1"` (selects 2nd scanned network)
      * `b_net2`: `prstr "W2"` (selects 3rd scanned network)
      * `b_net3`: `prstr "W3"` (selects 4th scanned network)
  * `b_rescan` (Button): `[ RESCAN ]` -> `prstr "N"`
  * `b_cancel` (Button): `[ < CANCEL ]` -> `page 0`

### Page: `wifi_keyb` (On-Screen Touch Keyboard)
* **Trigger**: Automatically opened when an SSID is selected from `wifi_scan`.
* **Components**:
  * `t_sel_ssid` (Text): Displays selected network name (e.g. `"SSID: MyHomeWiFi"`).
  * `t_pass` (Text): Real-time password input box with masking option (e.g. `"********"`).
  * **Touch QWERTY Keypad Matrix**:
    * Row 1 (Numbers): `1`, `2`, `3`, `4`, `5`, `6`, `7`, `8`, `9`, `0`
    * Row 2 (Top Letters): `Q`, `W`, `E`, `R`, `T`, `Y`, `U`, `I`, `O`, `P`
    * Row 3 (Mid Letters): `A`, `S`, `D`, `F`, `G`, `H`, `J`, `K`, `L`
    * Row 4 (Bot Letters & Symbols): `Shift`, `Z`, `X`, `C`, `V`, `B`, `N`, `M`, `!@#`
    * Row 5 (Action Controls): `[ SPACE ]`, `[ BACKSPACE ]`, `[ CLEAR ]`, `[ CONNECT ]`
  * **Nextion Component Scripting**:
    * Each letter button appends its character to `t_pass.txt`:
      ```text
      t_pass.txt=t_pass.txt+"A"
      ```
    * `[ BACKSPACE ]`:
      ```text
      // Trims the last character of t_pass.txt
      covx t_pass.txt,t_pass.txt,0,0
      ```
    * `[ CONNECT ]`:
      ```text
      // Transmits 'K' followed by the password string to Arduino
      prstr "K"
      prints t_pass.txt,0
      prstr "\r\n"
      ```

---

## 5. Unified Local Weather & Space Weather Pages

The weather subsystem combines **GPS-localized terrestrial weather** (current conditions + 5-day daily forecast via Open-Meteo) with **HF Space Weather & solar propagation metrics** (NOAA / Space Weather Prediction Center & HamQSL):

### Page: `wx_dashboard` (Dual Weather Interface)
* **Header**: `"GPS LOCAL & HF SPACE WEATHER"` | Location: Current GPS Coordinates / Maidenhead Grid (e.g. `EM28` Lawrence, KS).
* **Upper Section: Local Ground Weather (GPS-Localized)**:
  * `t_cur_temp`: Current Temperature (e.g. `65.3°F`)
  * `t_cur_hum`: Humidity, Barometric Pressure & Wind Speed (`Hum: 52% | 1018 hPa | Wind: 9 mph S`)
  * `t_cur_desc`: Sky Condition icon/text (`Sunny / Clear Sky`)
* **Middle Section: 5-Day Terrestrial Forecast Grid**:
  * 5 compact day cards displaying High / Low temperatures and sky icons:
    * `Day 1 (Mon)`: High 68°F / Low 48°F [Sunny]
    * `Day 2 (Tue)`: High 64°F / Low 45°F [Partly Cloudy]
    * `Day 3 (Wed)`: High 61°F / Low 42°F [Showers]
    * `Day 4 (Thu)`: High 65°F / Low 46°F [Sunny]
    * `Day 5 (Fri)`: High 70°F / Low 50°F [Clear]
* **Lower Section: HF Space Weather & Ionospheric Propagation**:
  * `t_sfi`: **Solar Flux Index (SFI)** (e.g. `154`) — Drives MUF and upper HF band opening (15m, 12m, 10m).
  * `t_ssn`: **Sunspot Number (SSN)** (e.g. `98`) — Measures active solar region activity.
  * `t_ak`: **Geomagnetic Indices**: `A-Index: 5` | `K-Index: 1` (`Quiet / Stable Ionosphere`).
  * `t_xray`: **Solar X-Ray Flare Level**: `B1.4` (`Normal / No D-Region Absorption`).
  * `t_hf_cond`: **Band Condition Forecast**:
    * `80m - 40m`: Good (Day & Night)
    * `30m - 20m`: Good (Day & Night)
    * `17m - 10m`: Good (Day) / Fair (Night)
* **Touch Controls**:
  * `[ REFRESH WX ]`: Transmits `'U'` over Serial1 to fetch fresh GPS and space weather feeds.
  * `[ < MAIN MENU ]`: Returns to Main Menu Hub (`page 1`).
