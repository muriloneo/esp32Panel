# ESP32-C6 UI Panel  - KOMMANDO
A Zigbee touchscreen control panel for Home Assistant

---

## Overview

**ESP32-C6 UI Panel** is a touchscreen smart-home control device built on the **ESP32-C6** using the **ESP-IDF framework**.  
The panel integrates directly with **Home Assistant** through **Zigbee2MQTT**, providing a fast and reliable way to control devices from a dedicated wall or desk display.

The interface presents **six configurable tiles** on a color LCD. Each tile can be mapped to a Home Assistant entity such as lights, switches, fans, or covers. Tapping a tile toggles the linked entity, and the tile color reflects the real-time state of the entity across the entire Home Assistant ecosystem.

Changes triggered elsewhere — voice assistants, physical switches, automations, or other dashboards — are immediately reflected on the panel.

The system is designed to be:

- Local-first
- Zigbee native
- Fast and responsive
- Fully configurable from Home Assistant
- Reliable even across restarts and network events

---

# System Architecture

The system consists of three components:

1. **ESP32 firmware**
2. **Zigbee2MQTT external converter**
3. **Home Assistant blueprint**

```

User Tap
│
▼
ESP32 UI Panel
│ Zigbee (OnOff command + custom cluster report)
▼
Zigbee2MQTT
│ MQTT event
▼
Home Assistant
│
Entity toggle
│
State update
▼
MQTT command back to panel
▼
Panel UI updates

```

This architecture ensures:

- Reliable click detection
- Instant state synchronization
- Protection against echo loops

---

# Hardware

| Component | Details |
|-----------|--------|
| SoC | ESP32-C6 (RISC-V, integrated Zigbee 802.15.4) |
| Display | ST7789 SPI LCD (172×320) |
| Touch Controller | CST816S capacitive touch |
| Zigbee Role | Router |
| Firmware Framework | ESP-IDF |
| UI Library | LVGL |

### Pin Configuration

| Function | Pin |
|--------|------|
| SPI CLK | GPIO6 |
| SPI MOSI | GPIO7 |
| SPI CS | GPIO10 |
| Display DC | GPIO11 |
| Display RST | GPIO3 |
| Backlight | GPIO2 |
| I2C SDA | GPIO0 |
| I2C SCL | GPIO1 |
| Touch RST | GPIO20 |

### Bus Speeds

| Bus | Speed |
|----|------|
| SPI | 40 MHz |
| I2C | 400 kHz |

---

# User Interface

The screen layout consists of a header and six tiles.

```

+-----------------------------+
|  Home Assistant             |
+--------------+--------------+
|    Tile 1    |    Tile 2    |
+--------------+--------------+
|    Tile 3    |    Tile 4    |
+--------------+--------------+
|    Tile 5    |    Tile 6    |
+--------------+--------------+

```

Each tile displays:

- Icon
- Friendly name
- State label (ON/OFF)

### Tile Visual States

| State | Appearance |
|------|------------|
| Unconfigured | Dark background, grey icon |
| Configured OFF | Dark blue background |
| Configured ON | Yellow background |

---

# Tile Behavior

When a tile is pressed:

1. The tile state toggles locally.
2. A Zigbee **OnOff command** is sent from the tile endpoint.
3. A custom cluster report containing the tile ID and state is also sent.
4. Zigbee2MQTT converts the event into MQTT.
5. Home Assistant toggles the linked entity.
6. Entity state changes are sent back to the panel to update the UI.

---

# Zigbee Design

Each tile has its own Zigbee endpoint.

| Endpoint | Tile |
|---------|------|
| ep1 | Tile 0 |
| ep2 | Tile 1 |
| ep3 | Tile 2 |
| ep4 | Tile 3 |
| ep5 | Tile 4 |
| ep6 | Tile 5 |

This design ensures reliable click detection because **the endpoint identifies which tile generated the event**.

### Custom Cluster

Cluster ID: `0xFC11`

| Attribute | Direction | Purpose |
|----------|-----------|---------|
| payload | HA → Panel | configuration commands |
| state | Panel → HA | tile click report |

---

# Configuration Commands

Home Assistant configures tiles using a simple command format.

### Configure Tile

```

C:<tile_id>:<icon>:<name>

```

Example:

```

C:0:bulb:Kitchen

```

### Update Tile State

```

S:<tile_id>:<state>

```

Example:

```

S:3:1

```

---

# Zigbee2MQTT Integration

An external converter exposes the panel to Zigbee2MQTT.

### Device Model

```

UI_Panel_C6

````

### MQTT Topics

| Topic | Direction | Purpose |
|------|-----------|---------|
| zigbee2mqtt/ESP32Panel | device → HA | panel events |
| zigbee2mqtt/ESP32Panel/set | HA → device | commands |
| zigbee2mqtt/ESP32Panel/action | device → HA | tile actions |

Example event:

```json
{
  "action": "on",
  "tile_index": 3,
  "action_endpoint": 4,
  "tile_action": "T:3:1"
}
````

---

# Home Assistant Blueprint

The blueprint allows configuration of each tile from the Home Assistant UI.

For every tile you can define:

* Icon
* Name
* Entity to control

Supported entity domains:

* `light`
* `switch`
* `fan`
* `cover`

---

# Persistence

Tile configuration is stored in **NVS** on the ESP32.

Namespace:

```
ui_panel
```

Keys:

```
tile_0
tile_1
tile_2
tile_3
tile_4
tile_5
```

This allows the panel to recover its configuration even if Home Assistant is temporarily unavailable.

---

# Network Behavior

If the Zigbee network is unavailable:

* Network steering starts automatically.
* Steering retries after failures.
* The panel reconnects automatically after a leave event.

The device operates as a **Zigbee router** and can accept child devices.

---

# Firmware Tasks

| Task        | Purpose           |
| ----------- | ----------------- |
| zb_task     | Zigbee stack loop |
| tile_report | sends tile events |
| lcd_task    | initializes UI    |

---

# Project Structure

```
main/
 ├─ main.c
 ├─ CMakeLists.txt

zigbee/
 └─ esp32_panel.js

homeassistant/
 └─ esp32_panel_tile.yaml

managed_components/
 ├─ lvgl
 ├─ esp_lvgl_port
 ├─ esp_zigbee_lib
 └─ esp_lcd_touch_cst816s
```

---

# Features

* Zigbee native integration
* Real-time entity synchronization
* Configurable UI tiles
* Persistent configuration
* LVGL touchscreen interface
* Fully local operation
* No cloud dependency

---

# Requirements

* ESP-IDF v6.0
* Zigbee2MQTT
* Home Assistant
* ESP32-C6 hardware with ST7789 display

---

# Future Improvements

Potential future capabilities include:

* Additional tile pages
* Dynamic icons
* Brightness control
* OTA updates
* Device grouping
* Native Home Assistant discovery

---

# License

GPL v3

---

# Acknowledgments

This project builds upon:

* ESP-IDF
* LVGL
* Zigbee2MQTT
* Home Assistant
