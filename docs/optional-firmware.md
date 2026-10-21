# Optional advanced firmware — Inswift ETH gateways

**Stock (official) firmware is the default** for ETH-MG21, ETH-MG24, ETH-52P, ETH-52P7, and PoE variants. The paths below are **optional** community-oriented packages for users who want ESPHome or Tasmota on the ESP32 host while keeping the same Zigbee TCP bridge pattern as stock (`tcp://<IP>:6638`, 115200 8N1, no flow control).

| Path | What it is |
|------|------------|
| [esphome/eth-mg/](../esphome/eth-mg/README.md) | ETH-MG21 / MG24 — ESPHome 0.1.8 — Ethernet + Wi-Fi fallback, Zigbee TCP bridge, Home Assistant API, Bluetooth Proxy |
| [esphome/eth-52p/](../esphome/eth-52p/README.md) | ETH-52P / 52P7 — ESPHome 0.1.1 — Ethernet + Wi-Fi fallback, Zigbee TCP bridge, Home Assistant API, Bluetooth Proxy |
| [tasmota/eth-mg/](../tasmota/eth-mg/README.md) | ETH-MG21 / MG24 — Tasmota **ZigbeeBridgePro** template — console backlog, GPIO map, TCP bridge on port 6638 |
| [tasmota/eth-52p/](../tasmota/eth-52p/README.md) | ETH-52P / 52P7 — Tasmota **ZigbeeBridgePro** template — console backlog, GPIO map, TCP bridge on port 6638 |

Official product folders (for example [esp32-ETH-MG24](../esp32-ETH-MG24/README.md) and [esp32-ETH-52P](../esp32-ETH-52P/README.md)) are unchanged and remain the supported stock firmware channel.

## Before you flash

- Replacing the ESP32 application removes the stock web UI and OTA path until you reflash stock or one of these optional images.
- The Zigbee chip (EFR32 or CC2652) is **not** updated by ESP32 OTA on the ESPHome builds; use PC tools over TCP 6638 after **Zigbee Enter Bootloader** when you need new coordinator firmware.
- Fallback Wi-Fi AP defaults (ESPHome builds), intentional for recovery:
  - ETH-MG: SSID `Inswift-ETH-MG`, password `inswift123`
  - ETH-52P: SSID `Inswift-ETH-52P`, password `inswift123`
- The BSL pin polarity differs: ETH-MG holds BSL **low** for normal run (Tasmota `Output Lo` on GPIO33); ETH-52P holds BSL **high** (Tasmota `Output Hi` on GPIO12). Do not copy one board's Tasmota template to the other.
- Ethernet PHY: **IP101** (`EthType 1` in Tasmota; `type: IP101` in ESPHome).
- Zigbee client settings differ by chip:

  | Gateway | Zigbee2MQTT `adapter` | ZHA radio type |
  |---------|-----------------------|----------------|
  | ETH-MG21 / MG24 (EFR32) | `ember` | EZSP |
  | ETH-52P / 52P7 (CC2652) | `zstack` | ZNP |

## Quick links

- ESPHome ETH-MG overview: [esphome/eth-mg/README.md](../esphome/eth-mg/README.md)
- ESPHome ETH-MG step-by-step guide: [esphome/eth-mg/OPERATION_GUIDE.md](../esphome/eth-mg/OPERATION_GUIDE.md)
- Prebuilt ESPHome ETH-MG binaries: [esphome/eth-mg/firmware/](../esphome/eth-mg/firmware/README.md)
- ESPHome ETH-52P overview: [esphome/eth-52p/README.md](../esphome/eth-52p/README.md)
- ESPHome ETH-52P step-by-step guide: [esphome/eth-52p/OPERATION_GUIDE.md](../esphome/eth-52p/OPERATION_GUIDE.md)
- Prebuilt ESPHome ETH-52P binaries: [esphome/eth-52p/firmware/](../esphome/eth-52p/firmware/README.md)
- Tasmota notes (ETH-MG): [tasmota/eth-mg/README.md](../tasmota/eth-mg/README.md)
- Tasmota notes (ETH-52P): [tasmota/eth-52p/README.md](../tasmota/eth-52p/README.md)

Hardware and stock documentation: [Inswift ETH wiki](https://github.com/Inswift-dev/inswift-eth/wiki).
