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

#### Side Gate Control
* **Microcontroller**: ESP32 (Wemos D1 Mini 32)
* **Configuration**: `configs/gate-control.yaml`

#### Swimming Pool Pump
* **Microcontroller**: ESP8266 (ESP-12E)
* **Configuration**: `configs/swimming-pool-pump.yaml`

---

## Secrets Management
All sensitive credentials (Wi-Fi passwords, API keys, and usernames) are kept in the `configs/secrets.yaml` file. 

To prevent leaking these credentials, a dummy template is tracked in Git, and the local file with real values is marked to be ignored by Git using the `skip-worktree` index flag:
```bash
git update-index --skip-worktree configs/secrets.yaml
```
