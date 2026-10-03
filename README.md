# ADV Wardriver

![Version](https://img.shields.io/badge/version-2.1.0--alpha-orange)
![Platform](https://img.shields.io/badge/platform-Cardputer%20ADV-blue)
![Firmware](https://img.shields.io/badge/firmware-UIFlow2%20v2.5.3-lightgrey)
![License](https://img.shields.io/badge/license-Apache%202.0-green)

**A pocket-sized WiFi wardriving tool for the M5Stack Cardputer ADV with the Cap LoRa-1262 (GNSS).**
Scans for access points, tags them with GPS coordinates and logs them straight to the SD card in WiGLE CSV format.

<p align="center">
  <img src="images/bootscreen.jpg" alt="ADV Wardriver boot screen" width="480">
</p>

---

## The story

On May 11th 2026 I started turning an M5Stack Tab5 into a fully fledged, standalone portable wardriving platform: the **[Tab5 Wardriver](https://github.com/Runaque/Tab5_Wardriver)**. I submitted it to the **M5Stack Global Innovation Contest 2026**, where it won a **Special Mentions – Wireless Innovation Award**.

With the gift card I won, I bought a Cardputer ADV and the Cap LoRa-1262, with one goal: port the Tab5 firmware to something that fits in a pocket. Within 20 hours of the ADV landing on my doorstep, ADV Wardriver was up and running. It was published on M5Burner the same day.

ADV Wardriver is based on Tab5 Wardriver v1.4.7, rebuilt for a much smaller device: a 240×135 screen instead of 1280×720, keyboard instead of touch, and about 110 KB of free RAM instead of 32 MB of PSRAM.

---

## Features

- **WiFi scanning** every 15 seconds (2.4 GHz)
- **GPS tagging** via the AT6668 GNSS on the Cap LoRa-1262 (GPS, GLONASS, BeiDou, Galileo, QZSS)
- **GPS-fix-only logging**: no zero-coordinate entries in your data
- **WiGLE CSV format**, ready to upload to [wigle.net](https://wigle.net)
  - real GPS UTC timestamps
  - WiGLE capability strings (`[WPA2-PSK][WPA3-SAE][ESS]`, …)
  - hidden networks logged with an empty SSID, as WiGLE expects
- **One log file per session** in a `Wardriver logs` folder on the SD card
- **POI waypoints** saved to a GPX file with one key press
- **Battery percentage** on screen, with automatic save of the session at a critical battery level
- **Flash fallback** (1 MB cap) when no SD card is present
- **Boots straight into the app**, with a UIFlow2 recovery menu one key away

---

## Hardware

| Part | Notes |
|------|-------|
| M5Stack Cardputer ADV | StampS3A (ESP32-S3FN8, 8 MB flash, no PSRAM) |
| M5Stack Cap LoRa-1262 | AT6668 GNSS used; SX1262 LoRa kept idle |
| microSD card | FAT32 |

> **Note:** this firmware is made for the **Cardputer ADV**. The original Cardputer uses a different keyboard and is not supported.

---

## Installation

### Option 1: M5Burner (easiest)

1. Open **M5Burner** and find **ADV Wardriver** in the Cardputer section
   *(or use **Share Burn** with a share code while the public listing is pending)*.
2. Put the Cardputer ADV in **download mode**: side switch **off**, hold **G0**, switch **on**, release G0.
3. Select the COM port and click **Burn**.
4. Switch the device off and on. It boots straight into ADV Wardriver.

### Option 2: Flash the .bin yourself

Download `ADV_Wardriver_vX.X.X-alpha.bin` from the [Releases](../../releases) page. It's a full 8 MB image, written at address `0x0`.

**Web flasher** (Chrome or Edge): open [esptool-js](https://espressif.github.io/esptool-js/), connect in download mode, set Flash Address to `0x0`, choose the `.bin` and click **Program**.

**Command line:**

```bash
pip install esptool
esptool --chip esp32s3 --port COM9 write_flash 0x0 ADV_Wardriver_v2.1.0-alpha.bin
```

> On Windows the Cardputer shows up on **two different COM ports**: one in download mode (for flashing) and one while running (for Thonny). Use the download-mode port for flashing.

### Option 3: Manual install on UIFlow2

For developers who want to tinker with the code:

1. Flash **UIFlow2** for the Cardputer-Adv with M5Burner.
2. Connect with [Thonny](https://thonny.org) (Interpreter: MicroPython (ESP32)) and press **Stop**.
3. Copy `src/main.py`, `src/boot.py` and `images/bootscreen.jpg` to `/flash` on the device.
4. Clear the stored boot setting so `boot.py` starts the app directly:

```python
import esp32; n = esp32.NVS("uiflow"); n.erase_key("boot_option"); n.commit()
```

5. Power-cycle the device.

The included `boot.py` is UIFlow2's own startup file with one change: the default `boot_option` is `0` (run `main.py` directly) instead of `1` (startup menu).

---

## Usage

### Controls

On the Cardputer ADV, hold **Ctrl** while pressing a key. The **G0** button works without Ctrl.

| Key | Action |
|-----|--------|
| **ENTER** (ok) | Start a session · press **twice** within 2 s to stop |
| **P** | Save a POI (GPS waypoint) |
| **+ / −** (or **; / .**) | Brightness up / down |
| **C** | Clear the display (when stopped) |
| **G0** button | Same as ENTER |

### The screen

```
ADV_Wardriver v2.1.0α     [SD]     87%
Logged:388   Scan:24   00:11:02
GPS: FIX  Sats 9/14
51.20714, 4.44947
SD ADV_Wardriver_003.csv 41.2KB
Scanning + logging (GPS fix)
────────────────────────────────────────
 -64  6 WPA2  Telenet-XXXX
 -72  1 WPA2  Proximus-Home-XXXX
 -75 11 WPA2/3 Orange-XXXX
 -81  6 OPEN  Guest-WiFi
ENT:Start/Stop P:POI +/-:Bri C:Clr
```

- **Logged**: unique APs written to the log this session
- **Scan**: APs seen in the latest scan
- **Sats used/in view**: the second number climbing means the antenna hears satellites, even before a fix

### Typical session

1. Switch on outdoors and wait for **GPS: FIX** (a first cold fix can take 5–15 minutes; later fixes are much faster).
2. Press **Ctrl + ENTER** to start. A new log file is created.
3. Walk, cycle or drive. Watch **Logged** go up.
4. Press **Ctrl + ENTER twice** to stop and save before switching off.
5. Upload the CSV from the SD card to [wigle.net](https://wigle.net/uploads).

---

## Logging

### Files

```
/sd/Wardriver logs/
├── ADV_Wardriver_001.csv
├── ADV_Wardriver_002.csv
├── …
└── ADV_Wardriver_POI.gpx
```

- Each session gets a new, numbered CSV.
- Rows are flushed to the card every 20 APs and on stop, so always stop a session before switching off.
- Without an SD card, logs go to `/flash/Wardriver logs/`, capped at 1 MB.

### WiGLE CSV format

```
WigleWifi-1.4,appRelease=2.1.0-alpha,model=ADV_Wardriver,release=2.1.0-alpha,device=ADV_Wardriver,display=240x135,board=CardputerADV,brand=Runaque
MAC,SSID,AuthMode,FirstSeen,Channel,Frequency,RSSI,CurrentLatitude,CurrentLongitude,AltitudeMeters,AccuracyMeters,Type
AA:BB:CC:DD:EE:FF,ExampleNet,[WPA2-PSK][ESS],2026-10-03 14:05:01,6,2437,-75,51.00000000,4.00000000,11.3,,WiFi
```

### Rules

- An AP is logged **once per session**, at the first moment it's seen **with a GPS fix**.
- APs seen before the first fix are **not** marked as seen, so they still get logged once the fix arrives.
- Timestamps are **GPS UTC** time.

---

## Battery and safety

- The battery percentage is shown at the top right (green > 50%, yellow > 20%, red below).
- At **3% or lower** for about 30 seconds, a running session is **stopped and saved automatically**, so you never lose your last rows to a dead battery.

> ⚠️ **Don't wardrive while charging.** Charging, scanning and a bright screen together in a small closed case generate a lot of heat. Charge beforehand, on a hard, non-flammable surface, and keep an eye on the device. If it gets too hot to hold, or the case looks swollen, stop using it.

---

## Recovery

Hold **ESC** while switching on to open the **UIFlow2 startup menu** without removing ADV Wardriver. This sets the menu as the default; to boot straight into ADV Wardriver again, run in Thonny:

```python
import esp32; n = esp32.NVS("uiflow"); n.set_u8("boot_option", 0); n.commit()
```

You can always reach the REPL over USB with Thonny by pressing **Stop**. The app closes its log file cleanly when interrupted.

---

## Technical notes

| Function | Pins / setting |
|----------|----------------|
| GPS UART | TX = G13, RX = G15, 115200 baud, UART1, 4 KB RX buffer |
| microSD | SPI slot 3: SCK = G40, MISO = G39, MOSI = G14, CS = G12 |
| SX1262 NSS | G5, held **high** at boot so LoRa stays off the shared SPI bus |
| Display | 240×135 ST7789, default 6×8 font |

**Designed for a no-PSRAM ESP32-S3:**

- Logs are written directly to SD, with no full-file copies in RAM.
- The dedup set stores MACs as small integer hashes (4,000-entry cap).
- NMEA is assembled from a buffer into complete lines, and only checksum-valid sentences are parsed.
- POIs are appended to the GPX file in place, with constant memory use.

**Measured on the Cardputer ADV:** about 110 KB free RAM, 812 KB/s SD write speed (identical with WiFi active), and a WiFi scan of about 2.5 s at about 1 KB RAM cost.

### Changes compared to Tab5 Wardriver v1.4.7

- Fixed: APs seen before the GPS fix were never logged during that session.
- Fixed: the security type mapping was off by one from code 5 upwards (WPA2-Enterprise was missing).
- Fixed: NMEA lines could be lost or merged under load (UART buffer overflow).
- Fixed: the GPX waypoint name tag (`<n>` → `<name>`).
- New: GPS UTC timestamps, WiGLE capability strings, and empty SSIDs for hidden networks.

---

## Field test

First real-world test, October 3rd 2026: **about 390 unique APs in about 11 minutes**, partly stationary and partly walking a residential block in Antwerp. That included a steady 15-second scan interval, a clean GPS track, and no gaps, duplicates or errors.

---

## Known issues

- Keys need **Ctrl** held on the Cardputer ADV (to be investigated).
- The 1-hour endurance test is still pending.
- Installation through Launcher is not tested yet.
- The battery percentage is new in 2.1.0 and still needs to be verified on hardware.

## Roadmap

- [ ] Keys without Ctrl
- [ ] 1-hour endurance test
- [ ] Launcher compatibility test
- [ ] Optional filter for moving hotspots (car WiFi, phone hotspots)

---

## Changelog

**v2.1.0-alpha**
- CSV and GPX files in a `Wardriver logs` folder
- Battery percentage in the title bar
- Automatic session save at a critical battery level
- Separator line fix

**v2.0.0-alpha** (2026-10-03)
- First release: port of Tab5 Wardriver v1.4.7 to the Cardputer ADV + Cap LoRa-1262
- Published on M5Burner

---

## Responsible use

ADV Wardriver only **passively records** information that access points broadcast publicly (network name, MAC address, security type, signal strength). It doesn't connect to networks, capture traffic or attempt to access anything. Know and respect the laws in your country, and be thoughtful about sharing data that can reveal where people live, including you.

---

## Credits

- **Author:** Dennis Dockx ([Runaque](https://github.com/Runaque)), Antwerp, Belgium
- **Built on:** [M5Stack UIFlow2 MicroPython](https://github.com/m5stack/uiflow-micropython) (MIT License, © M5Stack Technology Co., Ltd)
- **Predecessor:** [Tab5 Wardriver](https://github.com/Runaque/Tab5_Wardriver) · [Hackster write-up](https://www.hackster.io/Runaque/tab5-wardriver-a-custom-gps-enabled-wardriving-platform-d5948a)

## License

ADV Wardriver is licensed under the **Apache License 2.0**. See [LICENSE](LICENSE).

---

<p align="center">
  <i>A healthy wardriving session starts with a healthy breakfast.<br>
  Don't forget to pack a healthy snack and some water for a long session.</i><br><br>
  Built in Antwerp by Runaque
</p>
