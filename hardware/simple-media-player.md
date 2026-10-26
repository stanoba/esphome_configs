# Simple Media Player - Hardware Design & Wiring Guide

This document describes the hardware design, component pinout, and wiring instructions for the ESP32-S3 and PCM5102A DAC media player module, optimized for **Music Assistant** and **Home Assistant**.

---

## 1. Hardware Overview

### Microcontroller: ESP32-S3-DevKitC-1 v1.0
* **Module**: ESP32-S3-WROOM-1 / N16R8
* **SoC**: ESP32-S3 (QFN56 revision v0.2), Dual-core Xtensa® 32-bit LX7 @ 240 MHz
* **Flash Memory**: 16 MB SPI Flash
* **PSRAM**: 8 MB Octal SPI (OPI) PSRAM @ 80 MHz, 3.3 V
* **Interfaces**: Native USB-C (OTG) + CP2102 UART USB-C (dual USB-C ports)
* **Onboard LED**: WS2812 Addressable RGB LED on **GPIO 38**

> [!WARNING]
> **Octal PSRAM Pin Restrictions:**
> The ESP32-S3 N16R8 uses an 8-line Octal SPI interface. Pins **GPIO 33, 34, 35, 36, and 37** are reserved internally for the Octal PSRAM/Flash memory bus and **MUST NOT** be connected or used for external peripherals.
> Additionally, **GPIO 46** is a strapping pin and is **input-only** (cannot be used as an I2S clock or data output).

### Audio DAC: TI PCM5102A Breakout Module
* **Resolution / Sample Rate**: 32-bit, up to 384 kHz stereo audio DAC
* **Dynamic Range / SNR**: 112 dB SNR, -93 dB THD+N
* **Integrated Charge Pump**: Generates negative analog rails internally (ground-referenced 2.1 Vrms output, removing the need for bulky DC-blocking capacitors)
* **Onboard Regulator**: Low-dropout (LDO) 3.3 V regulator for clean analog supply filtering
* **Audio Output**: 3.5 mm stereo headphone jack + line-out solder pads (`L, G, R`)

---

## 2. Module Placement & Physical Orientation

To ensure compact assembly, minimal wire lengths, and practical enclosure integration:

1. **ESP32-S3-DevKitC-1**:
   - Oriented horizontally with the PCB antenna on the left.
   - Dual USB-C connectors (`UART` and `USB`) face to the **right**.
2. **PCM5102A DAC**:
   - Positioned directly beneath the bottom header row of the ESP32 board.
   - The 3.5 mm audio jack faces **downwards**.
   - The 6-pin I2S input header (`SCK, BCK, DIN, LCK, GND, VIN`) sits on the **left** edge.
   - The top auxiliary row (`1, 2, 3, 4, G, R, G, L`) faces **upwards** toward the ESP32 pins.

---

## 3. Wiring and Pin Mapping

Because the modules are positioned in close proximity, point-to-point solid core jumper wires can be used with minimal length (10 mm to 30 mm).

### Wiring Table

| Signal | PCM5102A Pin | ESP32-S3 Pin | Function / Description | Recommended Wire Length |
| :--- | :--- | :--- | :--- | :--- |
| **BCK** | Pin 2 (Side 6-pin) | **GPIO 8** | I2S Bit Clock (synchronizes data bits) | ~12–15 mm |
| **DIN** | Pin 3 (Side 6-pin) | **GPIO 18** | I2S Serial Data Out from ESP32 | ~12–15 mm |
| **LCK** | Pin 4 (Side 6-pin) | **GPIO 17** | I2S Word Select / Left-Right Clock | ~12–15 mm |
| **XSMT** | Pin 3 (Top row) | **GPIO 10** | Soft Mute control (HIGH = Unmute, LOW = Mute) | ~10 mm |
| **Jack Detect** | 3.5mm Socket switch | **GPIO 14** | Headphone/cable insertion detection | ~12–15 mm |
| **VIN** | Pin 6 (Side 6-pin) | **5V** (Pin 21) | 5 V power rail into the DAC's onboard LDO | ~25–30 mm |
| **GND** | Pin 5 (Side 6-pin) | **GND** (Pin 22) | Common ground | ~25–30 mm |
| **SCK** | Pin 1 (Side 6-pin) | *Unconnected* | Internal PLL mode (generates SCK from BCK) | – |

```
       ESP32-S3-DevKitC-1 (bottom header row near USB-C)
       +-------------------------------------------------------------------------------+
       | ... [6]  [7]   [15]  [16]  [17] [18] [8] [3] [46] [9]  [10]  [11] ... [5V] [GND]| === [USB-C 2x]
       +-----------|------|-----|---------------------------------|-----|---------|----|---+     (facing right)
                   |      |     |                                 |     |         |    |
                   |      |     |             +-------------------+     |         |    |
                   |      |     |             |   (XSMT Soft Mute)      |         |    |
                   |      |     |             |                         |         |    |
                   |      |     |             |        +----------------+         |    |
                   |      |     |             |        | (Jack Detection)         |    |
                   |      |     |             |        |                          |    |
             +-----|------|-----|-------------|--------|--------------------------|----|---+
             | [SCK]      |     |             |        |                          |    |   |
             |   [BCK] <--+     |             |        |                          |    |   |
             |     [DIN] <------+      [1] [2] [3] [4] [G] [R] [G] [L]            |    |   |
             |       [LCK] <----+       |   |   |   |                         |    |   |
             |         [GND] <----------------------------------------------------+    |   |
             |           [VIN] <-------------------------------------------------------+   |
             |                                              +-------------+                |
             |                PCM5102A DAC                  |  3.5mm JACK |                |
             |                                              |             |                |
             +----------------------------------------------+--[       ]--+----------------+
                                                               | AUDIO | (facing DOWN)
                                                               +-------+
```

---

## 4. Key Circuit Design Notes

### 1. Soft Mute (XSMT on GPIO 10)
* On breakout boards where the solder jumpers on the bottom are unbridged, the PCM5102A soft-mute line defaults to a low/muted state.
* By wiring **Pin 3 (XSMT)** on the top header to **GPIO 10**, ESPHome can dynamically control DAC muting.
* On boot, the DAC remains muted while the I2S clocks stabilize and network connects, preventing annoying "pops" or "thumps" in connected speakers or headphones. Once ready, GPIO 10 is driven `HIGH` to enable audio output.

### 2. Headphone Jack Detection (GPIO 11)
* Standard 3.5 mm PCB audio sockets feature an internal mechanical detection switch contact.
* Wiring this contact to **GPIO 11** with an internal pull-up resistor allows ESPHome to immediately detect whether headphones or an aux cable are plugged in:
  - **Plug Inserted**: Unmutes the DAC and enables playback.
  - **Plug Removed**: Automatically pauses/stops media playback in Music Assistant and mutes the DAC.

### 3. Power Supply (5V vs 3.3V)
* Connecting **VIN** to the ESP32-S3's **5V** pin (Pin 21) rather than the 3.3V pin allows the PCM5102A's onboard linear LDO regulator to regulate down to 3.3V cleanly.
* This isolates the sensitive analog audio circuitry from high-frequency switching noise produced by the ESP32-S3's Wi-Fi / Bluetooth power amplifier.

### 4. SCK / System Clock (Internal PLL)
* The PCM5102A incorporates an integrated phase-locked loop (PLL) that automatically derives the master system clock (`SCK`) directly from the bit clock (`BCK`).
* Leaving SCK unpopulated (or pulling it to GND) enables this PLL mode, eliminating the need to route a high-frequency MCLK line from the ESP32.

---

## 5. Next Steps
- Solder the prototype jumper wires according to the table above.
- Take updated high-resolution photos of the completed prototype.
- Draw the complete schematic in Fritzing and export vector/PNG files for this directory.
