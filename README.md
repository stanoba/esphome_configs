# ESPHome Configurations

This repository contains ESPHome configuration files for various smart home devices in my setup, including smart power plugs, water flow sensors, gate controls, and pool pumps.

---

## Hardware & Software Details

### 1. LSC Smart Plugs (Power Monitoring)
These smart plugs are based on the **LSC Smart Connect** smart plugs with power monitoring.
* **Product Link**: [LSC Smart Connect Smart Power Plug (Action)](https://www.action.com/sk-sk/p/3202088/inteligentna-zastrcka-s-monitorovanim-energie-lsc-smart-connect/)
* **Microcontroller**: Beken BK7231N (Tuya)
* **Framework**: [LibreTiny](https://libretiny.eu/)
* **Useful Documentation & Resources**:
  - [Elektroda: BK7231N Flashing & Teardown Guide](https://www.elektroda.com/news/news4087228.html)
  - [ESPHome Devices: LSC Plug With Monitoring Page](https://devices.esphome.io/devices/LSC-Plug-With-Monitoring)
  - [KeetSupport: How to Flash LSC Power Plug with ESPHome](https://keetsupport.nl/2024/03/20/how-to-flash-lsc-power-plug-with-esphome/)
  - [Tasmota Docs: LSC Smart Connect Smart Power Plug](https://tasmota.github.io/docs/devices/LSC-Smart-Connect-Smart-Power-Plug/)
* **3D Printed Flashing Device Jig**: [Thingiverse Jig by stanoba](https://www.thingiverse.com/thing:7132465)

#### Configurations:
- `configs/lsc-powerplug-pracka.yaml` (Washing Machine)
- `configs/lsc-powerplug-spirala.yaml` (Spiral Heating Element)
- `configs/lsc-powerplug-wifi-hore.yaml` (Upstairs WiFi Access Point - automatically scheduled ON in the morning and OFF in the evening)
- `configs/lsc-smartplug-umyvacka.yaml` (Dishwasher)
- `configs/lsc-powerplug-4.yaml` (LSC Smart Plug 4)
- `configs/lsc-smart-plug-5.yaml` (LSC Smart Plug 5 Nova)

---

### 2. Water Flow Sensors
These sensors measure water flow using a dedicated pulse meter and an LCD display.
* **Microcontroller**: [ESP32-C3 Super Mini](https://www.espboards.dev/esp32/esp32-c3-super-mini/)
* **Framework**: ESP-IDF

#### Configurations:
- `configs/garden-water-flow.yaml` (Garden Irrigation Flow)
- `configs/water-flow-sensor-tuv.yaml` (Hot Water Flow / TUV)

---

### 3. Other Devices

#### BlitzWolf Smart Plug
* **Use Case**: Automatically controls (ON/OFF) the circulation pump for hot water.
* **Product Details**: [BlitzWolf BW-SHP6 ESPHome Page](https://devices.esphome.io/devices/blitzwolf-bw-shp6/)
* **Configuration**: `configs/shp6.yaml`

#### Gate & Entrance Controls
* **Microcontroller**: ESP32 (Wemos D1 Mini 32 / ESP32 DevKit)
* **Configurations**:
  - `configs/gate-control.yaml` (Original Front Gate Control)
  - `configs/gate-control-2x.yaml` (Front Gate Control 2x / ESP-IDF)
  - `configs/side-gate-control-2.yaml` (Side Gate Control 2 / ESP-IDF)

#### Swimming Pool Pump
* **Microcontroller**: ESP8266 (ESP-12E)
* **Configuration**: `configs/swimming-pool-pump.yaml`

---

### 4. Mi Desk Lamp 1S (Standalone Smart Lamp)

A fully offline/standalone ESPHome conversion of the **Xiaomi Mi Desk Lamp 1S** (MJTD01SYL), physically a **Yeelight Mango1S Ver:0.3A** with an ESP32-WROOM-32D module inside.

* **Microcontroller**: ESP32-WROOM-32D (single-core variant with custom Yeelight eFuse MAC)
* **Framework**: ESP-IDF (required for single-core ESP32 and eFuse MAC workaround)
* **Configuration**: `configs/mi-desk-lamp.yaml`

#### Features
- **Fully standalone / offline** — WiFi is OFF by default, no cloud, no Home Assistant required
- **Triple-click knob** → toggles WiFi Access Point on/off (for web control)
  - 2 blinks = AP enabled, 1 blink = AP disabled
- **Single-click knob** → toggles lamp ON/OFF
- **Rotate knob** → adjusts brightness
- **Press + rotate knob** → adjusts color temperature (2500 K warm – 4800 K cool)
- **3× blink on boot**, then lamp turns OFF
- **Web interface** at `http://192.168.4.1/` (served locally, no internet needed)
  - Brightness slider, color temperature slider (K), ON/OFF toggle
  - ESPHome version and build timestamp shown
  - OTA firmware update via web UI
- **Blink feedback always visible** — works whether lamp is ON (blink-down) or OFF (blink-up)

#### WiFi / AP Usage
1. Triple-click the knob → lamp blinks 2× → AP `Mi-Desk-Lamp-AP` appears
2. Connect with the password hardcoded in the YAML
3. Open `http://192.168.4.1/` in your browser
4. Triple-click again → lamp blinks 1× → AP turns off

> **Note:** The captive portal is disabled intentionally — connecting to the AP will not auto-open a WiFi setup page. Navigate to `http://192.168.4.1/` manually.

#### GPIO Mapping

| Function | GPIO |
|---|---|
| Knob button | GPIO33 (INPUT_PULLDOWN, inverted) |
| Rotary encoder A | GPIO27 |
| Rotary encoder B | GPIO26 |
| Cold white LED (LEDC, 40 kHz) | GPIO2 |
| Warm white LED (LEDC, 40 kHz) | GPIO4 |
| LED PSU enable | GPIO12 |

#### Flashing

**First flash must be via USB** (partition table changes from factory Yeelight firmware):
```powershell
esphome run configs/mi-desk-lamp.yaml
```
Subsequent updates can be done via OTA through the web interface at `http://192.168.4.1/` (Choose File → `.bin` → Update), or via `esphome run` while AP is active.

#### Useful Resources
- [ESPHome CWWW Light](https://esphome.io/components/light/cwww.html)
- [ESPHome Rotary Encoder](https://esphome.io/components/sensor/rotary_encoder.html)
- [ESPHome Power Supply](https://esphome.io/components/power_supply.html)
- [ESPHome Web Server v3](https://esphome.io/components/web_server.html)

---

### 5. Simple Media Player (Audio Streamer)
A high-performance local network audio streamer optimized for **Music Assistant** and Home Assistant.
* **Microcontroller**: [ESP32-S3 DevKitC-1 (N16R8)](https://docs.espressif.com/projects/esp-idf/en/latest/esp32s3/hw-reference/index.html) (16 MB Flash, 8 MB Octal PSRAM)
* **DAC Module**: Texas Instruments PCM5102A I2S DAC (32-bit / 384 kHz stereo)
* **Framework**: ESP-IDF with Octal PSRAM and 512 kB stream buffer
* **Features**:
  - High-resolution audio streaming with zero stutter via Music Assistant (Sendspin)
  - Hardware pop suppression via Soft Mute control (XSMT on GPIO 10)
  - Headphone/cable insertion detection (GPIO 11) with auto-pause
  - Onboard WS2812 RGB LED playback status indication (GPIO 38)
* **Hardware Documentation**: See [hardware/simple-media-player.md](hardware/simple-media-player.md) for full pinout and wiring guide.
* **Configuration**: `configs/simple-media-player.yaml`

---

## Secrets Management
All sensitive credentials (Wi-Fi passwords, API keys, and usernames) are kept in the `configs/secrets.yaml` file. 

To prevent leaking these credentials, a dummy template is tracked in Git, and the local file with real values is marked to be ignored by Git using the `skip-worktree` index flag:
```bash
git update-index --skip-worktree configs/secrets.yaml
```
