# 📡 Remodaz Wifi

**Web-based universal IR remote, protocol analyzer & AC control — built with ESP32-C3 SuperMini**

<img width="1983" height="780" alt="13b40e0c-e108-4aec-a61a-226f0b6b80cb" src="https://github.com/user-attachments/assets/afe2f3be-a9e5-4e33-bef9-86643a1c4661" />



Version **v0.2.5** | DIY hardware project |

![ESP32-C3](https://img.shields.io/badge/MCU-ESP32--C3-blue?style=for-the-badge&logo=espressif)
![No Display](https://img.shields.io/badge/UI-Web%20Browser-orange?style=for-the-badge)
![IR 36kHz](https://img.shields.io/badge/IR-36kHz-red?style=for-the-badge)
![MicroSD](https://img.shields.io/badge/Storage-MicroSD-green?style=for-the-badge)
![AC Control](https://img.shields.io/badge/AC-70%2B%20Protocols-purple?style=for-the-badge)

ESP32-C3 runs its own Wi-Fi access point and serves the entire remote as a single web page. Connect your phone or laptop, and control your gear from the browser.

---
## Table of Contents

- [What Is This](#what-is-this)
- [At a Glance](#at-a-glance)
- [Connection](#connection)
- [Hardware](#hardware)
- [Wiring](#wiring)
- [Tab Layout](#tab-layout)
- [Feature Highlights](#feature-highlights)
- [SD Card Layout](#sd-card-layout)
- [Flash — Web Flasher](#flash--web-flasher)
- [Known Issues](#known-issues)
- [Project Phase](#project-phase)
  
## What Is This

Remodaz Wifi turns a bare ESP32-C3 SuperMini into a fully browser-driven universal IR remote: learn buttons from almost any remote, inspect and decode IR protocols with a built-in analyzer, and drive 70+ AC brands through a structured control panel — power, mode, temp, fan, swing, turbo, and more. Everything is stored on a microSD card and served over the ESP32's own captive-portal Wi-Fi network.

---

## At a Glance

| Capability | What it does |
| --- | --- |
| 🎛️ **Universal Remote** | Learn and replay IR buttons from almost any remote, from any browser |
| 🔬 **IR Analyzer** | Identify protocols, inspect raw timings, decode supported frames |
| ❄️ **AC Control** | Structured control (power/mode/temp/fan/swing/turbo/…) across 70+ AC brands |
| 💾 **SD Database** | Devices, key labels, raw captures, analyser exports |
| 🌐 **Web UI** | Served straight from the ESP32 — no app, no display, just connect and go |
| 🔁 **Hold / Repeat** | Real remote-style press-and-hold behaviour |
| 📶 **Captive Portal** | Joins as its own Wi-Fi AP; most phones pop the remote open automatically |

The remote stores the **raw IR timing waveform** for TV/universal keys, so replay never depends on the Analyzer knowing the protocol.

---
## Connection 
- Turn On your phone/laptop Wifi, join the **Remodaz Wifi** Wi-Fi network. 
- Most phones pop the remote open automatically (captive portal); if not, open `http://192.168.4.1` or `http://remodaz.local`.
- Default Wifi password **irremote123**
- Later default password can be changed by Clicking the Gear Icon, 
## Hardware

### Required

| Part | Notes |
| --- | --- |
| ESP32-C3 SuperMini board | 3.3V logic, USB-C |
| MicroSD card module (SPI) | 3.3V or 5V-tolerant breakout, SPI interface |
| MicroSD card | FAT32-formatted, 8–16GB recommended |
| IR receiver module | TSOP 1838 or equivalent 38kHz demodulated receiver |
| IR LED | 940nm, standard 5mm IR emitter |
| NPN transistor | e.g. S8050, to drive the IR LED from GPIO1 |
| Base resistor | **1kΩ**, transistor base → GPIO1 |
| Current-limiting resistor | For the IR LED, **100Ω** is recommended or **47Ω** can be used for better IR range. chosen LED's forward current/voltage |
| WS2812 / NeoPixel LED | Single pixel, for status feedback |
| Hookup wire, perfboard/protoboard | Point-to-point wiring, no custom PCB required |
| USB-C cable | Power + flashing |

For untethered, battery-powered operation instead of running off USB-C power directly:

| Part | Notes |
| --- | --- |
| 3.7V 800mAh Li-Po battery | Rechargeable pack |
| TP4056 5V/1A USB-C Li-Ion charging module | Charges the Li-Po pack over USB-C, with battery protection |
| 3.7V–12V mini DC-DC boost module | Steps the Li-Po's ~3.7V up to a stable 5V regulated rail for the board |
| AMS1117 3.3V DC-DC step-down module (optional) | Feeds the ESP32-C3 directly at 3.3V, bypassing its onboard regulator — see note below |

> Add a **0.1µF ceramic + 10µF electrolytic** decoupling capacitor pair at the boost module's output to smooth current surges into the board.
>
> If you see brownouts or frequent disconnects running off the boosted 5V rail, power the ESP32-C3 SuperMini's `3V3` pin directly from an AMS1117 3.3V step-down instead — some SuperMini boards have an unreliable onboard 5V→3.3V regulator, and feeding 3.3V straight in skips it.

⚠️ **3.3V logic only** — never feed 5V into a signal pin. `GPIO 9` is the ESP32-C3's boot-strapping pin; nothing in this build uses it, so don't repurpose it.

## Wiring

UI lives entirely in the browser. Everything below wires directly to the ESP32-C3 SuperMini.
<img width="450" height="270" alt="image" src="https://github.com/user-attachments/assets/88cc65c2-e616-453d-b95d-53048187cabe" />

### MicroSD Module

```
┌─────────────────┐              ┌──────────────────┐
│  MicroSD Module │              │   ESP32-C3       │
│                 │              │   SuperMini      │
├─────────────────┤              ├──────────────────┤
│ SCK ────────────┼──────────────┤ GPIO 4           │
│ MOSI ───────────┼──────────────┤ GPIO 6           │
│ MISO ───────────┼──────────────┤ GPIO 5           │
│ CS ─────────────┼──────────────┤ GPIO 10          │
│ VCC ────────────┼──────────────┤ 3V3              │
│ GND ────────────┼──────────────┤ G                │
└─────────────────┘              └──────────────────┘
```

### IR Receiver (TSOP)

```
┌─────────────────┐              ┌──────────────────┐
│   IR Receiver   │              │   ESP32-C3       │
│ (TSOP / VS1838B)│              │   SuperMini      │
├─────────────────┤              ├──────────────────┤
│ OUT ────────────┼──────────────┤ GPIO 3 (learn/rx)│
│ VCC ────────────┼──────────────┤ 3V3              │
│ GND ────────────┼──────────────┤ G                │
└─────────────────┘              └──────────────────┘
```

### IR Transmitter

```
┌─────────────────┐              ┌──────────────────┐
│ IR LED via      │              │   ESP32-C3       │
│ NPN driver      │              │   SuperMini      │
├─────────────────┤              ├──────────────────┤
│ Base (1kΩ) ─────┼──────────────┤ GPIO 1 (36kHz TX)│
│ Emitter ────────┼──────────────┤ G                │
│ Collector ──────┼──────────────| IR LED cathode   │
│                 │              | IR LED anode ──── R(100Ω) ─── 3V3 or 5V |
└─────────────────┘              └──────────────────┘
```
<img width="300" height="180" alt="image" src="https://github.com/user-attachments/assets/e8d18b33-fd9d-4aca-8e20-b8bad2cb050e" />

NPN transistor (e.g. S8050) driven through a base resistor **1KΩ**, rather than sourcing the IR LED current directly from the GPIO, **100Ω** or **47Ω** is recommended for better IR range.

### Status LED (WS2812 / NeoPixel)

```
┌─────────────────┐              ┌──────────────────┐
│   WS2812        │              │   ESP32-C3       │
│   NeoPixel      │              │   SuperMini      │
├─────────────────┤              ├──────────────────┤
│ DIN ────────────┼──────────────┤ GPIO 7           │
│ VCC ────────────┼──────────────┤ 5V               │    
│ GND ────────────┼──────────────┤ G                │
└─────────────────┘              └──────────────────┘
```
<div align="center">
  <img width="400" height="225" alt="WS2812" src="https://github.com/user-attachments/assets/5d279319-76f3-4f4b-850c-059386cba4d7" />
</div>

---
### Battery Power (optional)

```
┌──────────────┐      ┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│  Li-Po       │      │  TP4056      │      │  DC-DC Boost │      │  ESP32-C3    │
│  3.7V 800mAh │──────│  Charger     │──────│  Module      │──────│  SuperMini   │
│              │      │  (USB-C in)  │      │  (out: 5V+)  │      │    (5V)      │
└──────────────┘      └──────────────┘      └──────────────┘      └──────────────┘
```

Charge the Li-Po through the TP4056's USB-C port, feed its output into the boost module, and power the ESP32-C3 from the boost module's regulated output. Skip this section entirely if you're running the board straight off USB-C power.

<img width="700" height="400" alt="image" src="https://github.com/user-attachments/assets/872affba-d735-46c3-b5ce-517b66daeb6e" />

---

⚠️ **NOTE** — All Grounds must be shared in common.

## Tab Layout

```
REMODAZ WIFI (browser)
│
├── Devices                    Home tab — saved remotes
│   ├── + Add device           Create, reorder, rename, delete devices
│   └── <device>
│       ├── Key grid           Tap to transmit, hold to repeat
│       ├── + Key              Learn a new button
│       └── Edit Keys          Up/Down / Rename / Relearn / Delete
│
├── Analyzer                   Standalone capture + protocol decode
│   ├── Capture / Test / Save / Clear
│   └── Prev / Next            Browse last 5 captures (RAM only)
│
└── AC                         Structured AC remote control
    ├── Auto Detect            Learn an AC unit's protocol from its own remote
    ├── Change Device          Pick from 70+ supported AC protocols
    └── Edit                   Reorder the on-screen action buttons
```

Wi-Fi settings (SSID/password) and a reboot control live behind the gear icon in the header.

---

## Feature Highlights

### Universal Remote

Pick a device from the strip at the top of the **Devices** tab (or "+ Add device"), then tap keys to send them. Press and hold a key to repeat it, the same way a physical remote does. "Edit Keys" lets you select a key to rename, relearn, delete, or reorder it; "+ Key" learns a brand-new one.

**Learning a key:** name it → point the original remote at the receiver and press the button once → the app shows the listening indicator → **Test** / **Retry** / **Save** / **Cancel**. Replay works by storing each key's raw mark/space timings and gating the IR LED's 36kHz carrier to match — any protocol, decoded or not.

### IR Analyzer

Standalone tool, separate from the device database — the **Analyzer** tab. **Capture** listens for the next IR signal and shows its protocol (NEC, Samsung, Sony SIRC, RC5, RC6, Panasonic/Kaseikyo, JVC, or Unknown), a decoded address/command where possible, and the raw mark/space timings. **Test** replays it, **Save** writes it to `/REMOTE/CAPTURES/*.TXT` on the card, and Prev/Next step back through the last 5 captures.


### AC Remote Control

The **AC** tab drives structured AC control (power, mode, temp, fan, swing, turbo, quiet, eco, light, filter, clean, beep) across 70+ named AC protocols, via IRremoteESP8266's `IRac`/`stdAc` abstraction — the same library that powers the Analyzer.

- **Change Device** — pick your AC's protocol from the built-in library.
- **Auto Detect** — point the AC's own remote at the receiver and press a button; the firmware identifies the protocol and decodes the current state automatically.
- **Edit** — reorder which action buttons show first; the order is shared across every AC device (a device only shows the buttons it actually supports).
- A favorite AC device, once set, auto-loads after a reboot.

---

## SD Card Layout

FAT32-formatted. The remote still boots and lets you use the Analyzer without a card, but TV/universal device+key storage needs one.

```
/REMOTE/
├── DEVICE_ORDER.TXT
├── FAVORITE.TXT
├── D000/
│   ├── NAME.TXT
│   ├── KEY_ORDER.TXT
│   └── KEYS/
│       ├── K000     Raw pulse timings (uint16 count + pulses, µs)
└── CAPTURES/            IR Analyzer exports
```

A missing/corrupt order file only affects display order — the firmware falls back to physical folder scanning and rebuilds it. Limits: `MAX_DEVICES = 20`, `MAX_KEYS = 40` per device, `MAX_RAW_PULSES = 600`.

---

## Flash — Web Flasher

Flash straight from your browser. Nothing to install.

1. Grab the latest `firmware.bin` from the [Releases](../../releases) page.
2. Open [ESP Web Tools](https://esptool.spacehuhn.com/) in **Chrome or Edge** on desktop — Safari and phones can't flash.
3. Plug in the ESP32-C3 SuperMini with a data-capable USB-C cable and select its serial port when prompted.
4. Point the tool at `firmware.bin`, hit **Connect & Flash**, and power-cycle the board once it's done.

> A pre-built binary only works correctly if your wiring matches the [pin map](#wiring) above exactly — it has no way to know your GPIOs differ.
>
> ⚠️ **SD card:** FAT32-formatted, 8–16GB preferred — the firmware creates `/REMOTE/` and its contents automatically, nothing to pre-load.

| File | Flash Address |
| --- | --- |
| `bootloader.bin` | `0x0000` |
| `partition-table.bin` | `0x8000` |
| `firmware.bin` | `0x10000` |

[esp web tool flashing.webm](https://github.com/user-attachments/assets/750f3da9-22a3-44a5-8bed-c26be0a3583a)


### Troubleshooting

#### ESP32C3 Port Does Not Appear

Try:

- A different USB data cable
- A different USB port
- A different computer
- Installing the USB-to-serial driver required by your ESP32 board

#### Connection Fails

Try putting the ESP32 into bootloader mode:

**BOOT → RESET → Release BOOT** / **While Holding the Boot Button Connect the USB**

#### RemodazWifi Boots Incorrectly

Verify:

- The correct Remodaz firmware was selected.
- The firmware matches your ESP32 variant.
- The flash addresses match the firmware package.
- All required firmware files were flashed.

If necessary, erase the flash and perform a clean installation before flashing again.

### First Boot

On your phone/laptop, join the **Remodaz Wifi** Wi-Fi network. Most phones pop the remote open automatically (captive portal); if not, open `http://192.168.4.1` or `http://remodaz.local`.

Ships with zero devices — tap **+ Add device** to create your first one and start learning keys, via the web UI's Learn flow.

---

## Known Issues

| Issue | Fix |
| --- | --- |
| Device list is empty right after flashing | Expected — the firmware ships blank. Tap "+ Add device" to create your first one |
| A device you created earlier is missing | Check the SD card is FAT32 and wired correctly — enable `DEBUG_MODE 1` and check the serial monitor for `SD begin FAILED` |
| "No IR signal detected" while learning | Point the original remote directly at the receiver (GPIO 3), wait for a clean idle period, check its batteries |
| Key transmits but target ignores it | Inspect with the Analyzer → Capture; check the IR LED's polarity/resistor; test at close range |
| Can't reach `remodaz.local` | mDNS support varies by OS/router — use the AP's IP (`192.168.4.1`) instead |
| Rapid device switching feels laggy | Each switch reads from the SD card; avoid rapid-firing device taps if you're on a slow/large card |

---
### WEB Interface
[RemodazWifi.webm](https://github.com/user-attachments/assets/bcdc60d7-ff57-4e41-89cf-6ca476c8ff61)

## Project Phase

**v0.2.5** — Web UI firmware runs on ESP32-C3 SuperMini: SD-backed device/key database, IR learning with test-before-save, hold/repeat replay, IR Analyzer with protocol ID + decode, and a full AC control tab across 70+ protocols.

**[⬆ Back to top](#-remodaz-wifi)**
