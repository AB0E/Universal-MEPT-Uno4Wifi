# Nextion 2.8" HMI Screen Design & Code Specification
## Model: Nextion NX3224T024 (2.8-inch UART Touch Display, 320x240 Landscape)
### Station: AB0E | Grid: EM28 | Rig: W3PM Ultimate MEPT on Arduino UNO R4 WiFi

---

## 1. Project Overview & Hardware Interconnects

This document provides the complete, ready-to-implement screen design, component attributes, coordinate layouts, RGB565 color schemes, and event scripting for the **Nextion NX3224T024** display connected to the **Arduino UNO R4 WiFi**.

* **Display Model**: Nextion NX3224T024 (320x240 resolution, 4MB Flash, 3584B RAM)
* **Serial Interface**: Hardware `Serial1` on Arduino UNO R4 WiFi
  * **Nextion TX (Blue wire)** -> **Arduino D0 / RX1**
  * **Nextion RX (Yellow wire)** -> **Arduino D1 / TX1**
  * **Nextion +5V (Red wire)** -> **+5V Rail** (5V 1A power rail)
  * **Nextion GND (Black wire)** -> **Common Ground**
* **UART Parameters**: 9600 baud, 8 data bits, no parity, 1 stop bit (`9600, 8, N, 1`)
* **Termination**: Standard Nextion 3-byte delimiter (`0xFF 0xFF 0xFF` / `255 255 255`)

---

## 2. Nextion Editor Project Configuration

1. **New Project**: File -> New -> Save as `W3PM_MEPT_AB0E.HMI`.
2. **Device Selection**:
   * Model Family: **Basic**
   * Model: **NX3224T024_011** (or `NX3224T024_011R`)
3. **Display Direction**:
   * Orientation: **90 - Horizontal / Landscape**
   * Resolution: **320 width x 240 height**
   * Character Encoding: **ASCII** (or `iso-8859-1`)

### Font Generation (Tools -> Font Creator)
Generate and add the following 3 fonts to the **Fonts** pane:
* **Font 0 (ID 0)**: `Consolas 16` (Height: 16px, ASCII 32-126) - Button labels, sub-headings, diagnostic alerts.
* **Font 1 (ID 1)**: `Consolas 24 Bold` (Height: 24px, ASCII 32-126) - Live telemetry data (`t0` UTC Clock, `t1` Band/Freq, `t2` Power/SWR).
* **Font 2 (ID 2)**: `Segoe UI 20 Bold` (Height: 20px) - Header titles (`AB0E`, `EM28`, mode banners).

---

## 3. Color Palette (16-bit RGB565)

| Color Name | Hex Code | Decimal (Nextion) | Usage in UI |
| :--- | :---: | :---: | :--- |
| **Deep Navy (Background)** | `0x001F` | **31** | Canvas background (`cls 31`) |
| **Slate Navy (Card Panel)** | `0x1082` | **4226** | Telemetry card containers |
| **Header Slate** | `0x0421` | **1057** | Top navigation header bar |
| **Pure White** | `0xFFFF` | **65535** | Primary text labels & borders |
| **Electric Cyan** | `0x07FF` | **2047** | Frequency, RF mode, Locator |
| **Amber Warning** | `0xFD20` | **64800** | SWR warnings, Tx power alert |
| **Matrix Green** | `0x07E0` | **2016** | GPS Lock, Safe SWR, Active LPF |
| **Slate Gray (Muted)** | `0x8410` | **33808** | Grid lines, secondary captions |
| **Button Base Blue** | `0x2965` | **10597** | Navigation buttons (PB1, PB3, PB4) |
| **Button Select Teal** | `0x1CA7` | **7327** | PB2 Select button |
| **Button Save Green** | `0x05E0` | **1504** | SAVE (2+3) calibration button |
| **Button Menu Crimson** | `0xC800` | **51200** | MENU / Mode toggle button |
| **Button Pressed Yellow** | `0xFFE0` | **65504** | Active touch highlight |

---

## 4. Page 0 (`main`): Primary Telemetry & Touch Dashboard

### Component Attributes Table (Page 0)

| ID | Object Name | Type | X | Y | Width | Height | Font | BCO (Fill) | PCO (Text) | Default Text / Attributes |
|:--:|:-----------|:----:|:-:|:-:|:-----:|:------:|:----:|:----------:|:----------:|:-------------------------|
| 0 | `page0` | Page | 0 | 0 | 320 | 240 | - | 31 (Navy) | - | Page Name: `main` |
| 1 | `t_call` | Text | 6 | 4 | 70 | 22 | 2 | 1057 | 65535 | `AB0E` |
| 2 | `t_grid` | Text | 80 | 4 | 60 | 22 | 0 | 1057 | 2047 | `EM28` |
| 3 | `t_title` | Text | 144 | 4 | 110 | 22 | 0 | 1057 | 33808 | `W3PM MEPT` |
| 4 | `b_relays` | Button | 258 | 3 | 58 | 24 | 0 | 10597 | 65535 | `LPF >` |
| 5 | `p_card` | Picture/Hotspot | 4 | 32 | 312 | 112 | - | 4226 | 10597 | Border card background |
| **10** | **`t0`** | **Text** | **8** | **36** | **304** | **25** | **1** | **4226** | **65535** | `23:16:48 UTC GPS` (txt_maxl=30) |
| **11** | **`t1`** | **Text** | **8** | **63** | **304** | **25** | **1** | **4226** | **2047** | `WSPR 20m 14.097100` (txt_maxl=30) |
| **12** | **`t2`** | **Text** | **8** | **90** | **304** | **25** | **1** | **4226** | **64800** | `Tx: 7dBm  SWR: 1.15` (txt_maxl=30) |
| **13** | **`t3`** | **Text** | **8** | **117** | **304** | **23** | **0** | **4226** | **2016** | `LPF: 20m [RELAY ON]` (txt_maxl=30) |
| 14 | `j0` | Progress | 8 | 142 | 304 | 5 | - | 31 | 2047 | `val=0` (Tx cycle countdown) |
| **6** | **`b0`** | **Button** | **4** | **150** | **102** | **41** | **0** | **10597** | **65535** | `PB1  -` |
| **7** | **`b1`** | **Button** | **109** | **150** | **102** | **41** | **0** | **7327** | **65535** | `PB2 SEL` |
| **8** | **`b2`** | **Button** | **214** | **150** | **102** | **41** | **0** | **10597** | **65535** | `PB3 NXT` |
| **9** | **`b3`** | **Button** | **4** | **194** | **102** | **42** | **0** | **10597** | **65535** | `PB4  +` |
| **10** | **`b4`** | **Button** | **109** | **194** | **102** | **42** | **0** | **1504** | **65535** | `SAVE 2+3` |
| **11** | **`b5`** | **Button** | **214** | **194** | **102** | **42** | **0** | **51200** | **65535** | `MENU` |

---

## 5. Nextion Event Code (Copy-Paste Scripts)

### Page 0 Pre-Initialization Event (`page0.preinit`):
```text
bkcmd=0
sendxy=1
```

### Page 0 Post-Initialization Event (`page0.postinit`):
```text
prstr "READY:AB0E\r\n"
```

### Button Touch Press Events:
* `b0` (PB1 -): `prstr "1"`
* `b1` (PB2 SEL): `prstr "2"`
* `b2` (PB3 NXT): `prstr "3"`
* `b3` (PB4 +): `prstr "4"`
* `b4` (SAVE 2+3): `prstr "S"`
* `b5` (MENU): `prstr "M"`
* `b_relays` (Header button): `page 1`
* `b_back` (Page 1 return button): `page 0`

---

## 6. MicroSD Card Flashing Instructions

1. Format a MicroSD card (32GB or smaller) with **FAT32** (4096-byte allocation).
2. In Nextion Editor, click **Compile**. Verify 0 Errors.
3. Select **File -> TFT file output**, and save the `.tft` file directly to the root of the MicroSD card.
4. Power off the Nextion display.
5. Insert the card into the Nextion card slot.
6. Apply +5V power. Wait for `Update Successed!`.
7. Power off, remove the card, and re-power the display.
