# Optional advanced firmware — Inswift ETH-MG gateways

**Stock (official) firmware is the default** for ETH-MG21, ETH-MG24, and PoE variants. The paths below are **optional** community-oriented packages for users who want ESPHome or Tasmota on the ESP32 host while keeping the same Zigbee TCP bridge pattern as stock (`tcp://<IP>:6638`, 115200 8N1, no flow control).

| Path | What it is |
|------|------------|
| [esphome/eth-mg/](../esphome/eth-mg/README.md) | ESPHome 0.1.8 — Ethernet + Wi-Fi fallback, Zigbee TCP bridge, Home Assistant API, Bluetooth Proxy |
| [tasmota/eth-mg/](../tasmota/eth-mg/README.md) | Tasmota **ZigbeeBridgePro** template — console backlog, GPIO map, TCP bridge on port 6638 |

Official product folders (for example [esp32-ETH-MG24](../esp32-ETH-MG24/README.md)) are unchanged and remain the supported stock firmware channel.

## Before you flash

- Replacing the ESP32 application removes the stock web UI and OTA path until you reflash stock or one of these optional images.
- The EFR Zigbee coordinator is **not** updated by ESP32 OTA on the ESPHome build; use PC tools over TCP 6638 after **Zigbee Enter Bootloader** when you need a new NCP `.gbl`.
- Fallback Wi-Fi AP defaults (ESPHome build): SSID `Inswift-ETH-MG`, password `inswift123` — intentional for recovery.
- Ethernet PHY: **IP101** (`EthType 1` in Tasmota; `type: IP101` in ESPHome).

## Quick links

- ESPHome overview: [esphome/eth-mg/README.md](../esphome/eth-mg/README.md)
- Step-by-step ESPHome guide: [esphome/eth-mg/OPERATION_GUIDE.md](../esphome/eth-mg/OPERATION_GUIDE.md)
- Prebuilt ESPHome binaries: [esphome/eth-mg/firmware/](../esphome/eth-mg/firmware/README.md)
- Tasmota notes: [tasmota/eth-mg/README.md](../tasmota/eth-mg/README.md)

Hardware and stock documentation: [Inswift ETH wiki](https://github.com/Inswift-dev/inswift-eth/wiki).
