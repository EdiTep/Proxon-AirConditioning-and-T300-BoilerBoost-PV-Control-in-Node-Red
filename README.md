[![Logo edi.teppert.com Blog](pictures/Edis-Blog-IoT-Logo.png)](pictures/Edis-Blog-IoT-Logo.png)

# Proxon Air Conditioning & T300 Boiler Boost with PV Surplus Control in Node-RED

[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)
[![Node-RED](https://img.shields.io/badge/Node--RED-3.0+-red.svg)](https://nodered.org/)
[![Dashboard 2.0](https://img.shields.io/badge/Dashboard-2.0-0094CE.svg)](https://dashboard.flowfuse.com/)
[![Modbus RTU](https://img.shields.io/badge/Modbus-RTU%20RS485-6E4C13.svg)](https://flows.nodered.org/node/node-red-contrib-modbus)
[![Powered by MQTT](https://img.shields.io/badge/Powered_by-MQTT-660066.svg?logo=mqtt&logoColor=white)](https://mosquitto.org/)
[![Powered by Shelly](https://img.shields.io/badge/Powered_by-Shelly-4495D1.svg)](https://www.shelly.com/)
[![Powered by Pushover](https://img.shields.io/badge/Powered_by-Pushover-0088FF.svg?logo=android&logoColor=white)](https://pushover.net/)

[![Klimaanlage und Boiler Boost Titel](pictures/2026-10-20-edi-teppert-com-Klimaanlage-BoilerBoost-PV-Titel-2048x820.jpg)](pictures/2026-10-20-edi-teppert-com-Klimaanlage-BoilerBoost-PV-Titel-2048x820.jpg)

## 🌍 Language Selector / Sprachauswahl

| Language   | Documentation                  |
| ---------- | ------------------------------ |
| 🇬🇧 English | [README.md](README.md)         |
| 🇩🇪 Deutsch | [README.de.md](README.de.md)   |

Use **free solar power** for cooling and hot water. This Node-RED flow switches the **cooling function of a Proxon heat pump ventilation unit** and the **Boiler Boost of a Proxon T300 hot water heat pump** automatically – but only when the PV system delivers enough power, the home battery is sufficiently charged and (for cooling) it is actually warm outside.

All values come from an **E3/DC home power station via MQTT** and directly from the **Proxon units via Modbus RTU (RS485)**. Every switching action is reported via **Pushover**, and a **Node-RED Dashboard 2.0** page shows all values and lets you adjust the thresholds with sliders.

> **Note on the Proxon units:** The Modbus interface uses **19200 baud, 8 data bits, parity *even*, 1 stop bit**. With parity *none* you will not get any answers.

## Features

- **PV-controlled cooling (air conditioning)**
  - Switches on only if *outside temperature* **>** threshold **AND** *battery SOC* **≥** threshold **AND** *PV power* **>** threshold.
  - Writes the operating mode (holding register 16) and the cooling release (holding register 62) of the Proxon unit via Modbus.
  - Outside temperature from the unit's own **T3 fresh air sensor** (input register 198).
  - If no temperature value has been received yet, cooling stays **off** for safety.
- **PV-controlled Boiler Boost (T300)**
  - Switches a **Shelly Plus 1** via MQTT when *battery SOC* **≥** threshold **AND** *PV power* **≥** threshold – hot water from free electricity only.
- **Hysteresis / compressor protection**
  - Cooling: minimum on-time **15 min**, minimum off-time **10 min**.
  - Boiler Boost: minimum on-time **10 min**, minimum off-time **10 min**.
- **Pushover notifications** for every automatic switch-on / switch-off, including the current values and thresholds.
- **Node-RED Dashboard 2.0**
  - Gauges: outside temperature, PV power, house consumption, battery SOC.
  - Status tiles: heat pump operating mode (Off / Summer / Winter / ECO), cooling function, Boiler Boost.
  - Sliders for all thresholds; slider values are restored automatically when the dashboard reconnects.
- **Generic Modbus decoder** with a complete register map of the Proxon units (unit 41 and unit 20) – easy to extend for your own values.

## Prerequisites

- **Hardware**:
  - Raspberry Pi (or any server running Node-RED)
  - USB-to-RS485 adapter (e.g. on `/dev/ttyUSB0`) connected to the Proxon Modbus bus
  - Proxon heat pump ventilation unit with cooling function (Modbus unit **41**)
  - (Optional) Proxon T300 hot water heat pump (Modbus unit **20**)
  - Shelly Plus 1 for the T300 Boiler Boost input (MQTT enabled)
  - E3/DC home power station (or any PV system publishing its values via MQTT)
- **Software**:
  - [Node-RED](https://nodered.org/) (version 3.0 or later)
  - MQTT broker (e.g. [Mosquitto](https://mosquitto.org/))
  - Node-RED palettes:
    - `node-red-contrib-modbus`
    - `node-red-contrib-pushover`
    - `@flowfuse/node-red-dashboard`
  - Account (optional): [Pushover](https://pushover.net/)

[![Node-RED Flow Klimaanlage und Boiler Boost](pictures/2026-10-20-edi-teppert-com-Node-Red-Flow-Klimaanlage-BoilerBoost-1024x600.jpg)](pictures/2026-10-20-edi-teppert-com-Node-Red-Flow-Klimaanlage-BoilerBoost-1024x600.jpg) [![Node-RED Dashboard 2.0 Klimaanlage und Boiler Boost](pictures/2026-10-20-edi-teppert-com-Node-Red-Dashboard-2.0-Klimaanlage-BoilerBoost-785x1024.jpg)](pictures/2026-10-20-edi-teppert-com-Node-Red-Dashboard-2.0-Klimaanlage-BoilerBoost-785x1024.jpg)

## Data Sources

### MQTT topics (E3/DC and Shelly)

| Topic                                          | Content                         | Used for                           |
| ---------------------------------------------- | ------------------------------- | ---------------------------------- |
| `Haus/Keller/e3dc/solar/power`                 | PV power in W                   | Cooling + Boiler Boost condition   |
| `Haus/Keller/e3dc/battery/soc`                 | Battery state of charge in %    | Cooling + Boiler Boost condition   |
| `Haus/Keller/e3dc/home/power`                  | House consumption in W          | Dashboard display                  |
| `Haus/Keller/Shelly/PVBoiler/command/switch:0` | `on` / `off` (retained)         | Switching + status of Boiler Boost |

### Modbus registers (Proxon, unit 41)

| Register        | Type    | Meaning                                         | Access      |
| --------------- | ------- | ----------------------------------------------- | ----------- |
| 16              | Holding | Operating mode: 0 = Off, 1 = Summer, 2 = Winter, 3 = ECO | read / write |
| 62              | Holding | Cooling release: 0 = Off, 1 = On                | read / write |
| 198             | Input   | T3 fresh air (= outside temperature), scale 1/100 °C | read  |

The function node `Register decodieren` contains the complete register map (name, scaling, offset, unit, signed) for unit 41 and unit 20. Polling is done in three blocks (holding 16–22, holding 62–70, input 195–223) every **5 minutes**.

## Node-RED Flow

The flow consists of four main parts:

1. **Modbus polling & decoding**: Reads the Proxon registers, decodes them with the register map and extracts operating mode, cooling state and outside temperature.
2. **Collecting values**: Stores PV power, battery SOC, outside temperature and all slider thresholds in the flow context.
3. **Cooling control (hysteresis)**: Checks the conditions on every new value, writes holding registers 16 and 62 via Modbus and sends a Pushover message.
4. **Boiler Boost control (hysteresis)**: Checks battery and PV thresholds, switches the Shelly via MQTT and sends a Pushover message.

### Import the Flow

Copy the entire JSON array from [`flows/klimaanlage-boilerboost-pv-flow.json`](flows/klimaanlage-boilerboost-pv-flow.json) and paste it into the Node-RED editor (Menu → Import).

> ⚠️ **Important:** You must configure the following nodes with your own settings before the flow will work:
>
> - **MQTT broker**: Host, port and credentials of your broker.
> - **MQTT topics**: Adapt the E3/DC and Shelly topics to your installation.
> - **Modbus client**: Serial port (e.g. `/dev/ttyUSB0`), 19200 baud, parity **even**.
> - **Pushover**: Application key and user key.
> - **Dashboard**: Select your own `ui-base` / `ui-page` if you already have a dashboard.

## Configuration Steps

### 1. Configure the Modbus client

- Edit the `modbus-client` configuration node `Proxon Modbus USBtoRS485`.
- Set the serial port of your RS485 adapter and check: **19200 baud, 8 data bits, parity even, 1 stop bit, RTU-BUFFERED**.
- Unit IDs: **41** for the ventilation / heat pump unit, **20** for the T300.

### 2. Configure MQTT

- Edit the `mqtt-broker` configuration node and enter your broker address.
- Adapt the topics in the `mqtt in` nodes (`Verbrauch Watt`, `Einspeisung Watt`, `Akku Ladung in %`, `PV-Boiler Boost Status`).
- In the function node `Boiler-Boost pruefen & schalten (Hysterese)`, adapt the Shelly command topic in the line `msgShelly = { topic: ... }`.

### 3. Set up notifications

- Double-click one of the `pushover api` nodes and select or create the `pushover-keys` configuration with your application key and user key.

### 4. Adjust thresholds (Dashboard)

| Slider                                  | Default | Range        |
| --------------------------------------- | ------- | ------------ |
| Cooling – battery threshold (%)         | 90      | 0 – 100      |
| Cooling – PV power threshold (W)        | 1200    | 0 – 5000     |
| Cooling – outside temperature (°C)      | 25      | 15 – 35      |
| Boiler Boost – battery threshold (%)    | 50      | 20 – 100     |
| Boiler Boost – PV power threshold (W)   | 800     | 150 – 3000   |

The defaults are used until a slider has been moved for the first time.

### 5. Customize (Optional)

- **Hysteresis**: Change `MIN_EIN_MS` and `MIN_AUS_MS` in the two function nodes.
- **Polling rate**: Change the rate in the three `modbus-read` nodes (currently 5 minutes).
- **Additional values**: Every register from the register map is available in the output of `Register decodieren` as `u<unit>_<holding|input>_<address>`.

## Troubleshooting

- **No Modbus answers / timeouts**: Check parity (**even**!), baud rate, A/B wiring of the RS485 bus and the serial port name.
- **Cooling never switches on**: The outside temperature must be known. Check that input register 198 is read (block input 195–223) and look at the status text of the function node (`Temp. unbekannt` / `zu kalt`).
- **Switching seems delayed**: That is the hysteresis. Within the minimum on-/off-time the node shows `(Hysterese aktiv)`.
- **Slider values lost after a Node-RED restart**: The thresholds are cached in the global context. Configure a persistent `contextStorage` (e.g. `localfilesystem`) in `settings.js` to keep them across restarts.
- **Unit switches completely off**: When cooling is switched off, the flow writes operating mode **0 = Off** to holding register 16. If your unit should return to another mode (e.g. ECO), change the value in the function node `AC-Bedingung pruefen & schalten`.

## Files in this Repository

- `README.md` – This documentation (English).
- `README.de.md` – Documentation in German.
- `flows/klimaanlage-boilerboost-pv-flow.json` – The exported Node-RED flow.
- `pictures/2026-10-20-edi-teppert-com-Klimaanlage-BoilerBoost-PV-Titel-2048x820.jpg` – Picture in this repo
- `pictures/2026-10-20-edi-teppert-com-Node-Red-Flow-Klimaanlage-BoilerBoost-1024x600.jpg` – Picture in this repo
- `pictures/2026-10-20-edi-teppert-com-Node-Red-Dashboard-2.0-Klimaanlage-BoilerBoost-785x1024.jpg` – Picture in this repo
- `pictures/Edis-Blog-IoT-Logo.png` – Picture in this repo

## Flow Structure (Simplified)

```
[modbus-read U41 holding 16-22 / 62-70 / input 195-223]  (every 5 min)
    │
    ▼
[Register decodieren] ──► [Extract heat pump / cooling status]
                                   │
                                   ├──► [Dashboard: operating mode, cooling, outside temp.]
                                   └──► (outside temperature → flow context)

[MQTT E3/DC: PV power, battery SOC, house consumption]
[Dashboard sliders: thresholds] ──► (flow context + global cache)
    │
    ├──► [Cooling check & switch (hysteresis 15/10 min)]
    │         ├──► [modbus-write U41 holding 16: operating mode]
    │         ├──► [modbus-write U41 holding 62: cooling release]
    │         └──► [Pushover]
    │
    └──► [Boiler Boost check & switch (hysteresis 10/10 min)]
              ├──► [MQTT → Shelly Plus 1 (T300 Boiler Boost)]
              └──► [Pushover]

[ui-control: dashboard connect] ──► [Restore slider values]
```

## Contributing

Found a bug or have an idea for improvement? Feel free to open an issue or submit a pull request. Ideas for enhancements: using house consumption for an "AC is running" detection, a periodic reminder when the conditions are no longer met, or supporting other PV systems.

## 📫 Contact & Collaboration

I'm excited about **exchange, feedback, and collaboration** – especially on topics like:

- Energy management (PV, battery SOC, heat pumps)
- Node-RED flow optimization
- Integration of proprietary systems (e.g., Tahoma, Proxon)

You can best reach me via **email** (provided on my profile).

## License

This project is open-source and available under the [GNU General Public License v3.0](LICENSE).

Original blog post (German): [Klimaanlage und Boiler Boost mit PV-Steuerung in Node-RED](https://edi.teppert.com/TODO-LINK-ZUM-BLOGBEITRAG)

Author: Edi Teppert
