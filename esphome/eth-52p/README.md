# Inswift ETH-52P — Optional ESPHome Firmware

**Version:** 0.1.1  
**Target:** ESPHome 2026.9.1 (esp-idf)  
**Products:** ETH-52P (CC2652P2) · ETH-52P7 (CC2652P7)

This package is an **optional advanced firmware** for Inswift ETH-52P / ETH-52P7 Ethernet Zigbee gateways.  
**Stock (official) firmware remains the default** and is fully supported. Use this build if you want ESPHome integration, Home Assistant Bluetooth Proxy, a documented pin map, and the same Zigbee TCP bridge as stock (`tcp://IP:6638`).

> Status: 0.1.1 was verified on ETH-52P hardware on 2026-10-21 in Home Assistant — **ZHA** (ZNP), **Zigbee2MQTT** (`zstack`) and the **ESPHome** integration all working over Ethernet. The **Zigbee Enter Bootloader** button is included, but flashing the CC2652 coordinator over TCP 6638 has not yet been verified on this build. A Tasmota ZigbeeBridgePro template was also verified on ETH-52P on 2026-10-21 (Z2M and ZHA OK) — see [tasmota/eth-52p/](../../tasmota/eth-52p/README.md).

---

## Supported models

| Model | Zigbee SoC | How the firmware identifies it |
|-------|------------|--------------------------------|
| ETH-52P | CC2652P2 | Not auto-detected (see below) |
| ETH-52P7 | CC2652P7 | Not auto-detected (see below) |

Both models share the same ESP32 pinout and use the same YAML / binaries. There is no GPIO strap that tells P2 from P7, so the **Zigbee Chip** diagnostic sensor shows the fixed text `CC2652P2 / CC2652P7`. Zigbee2MQTT and ZHA report the actual coordinator chip after they connect.

---

## Features (vs stock)

| Feature | This firmware | Stock |
|---------|---------------|-------|
| Ethernet primary networking | Yes (ESPHome `type: IP101`) | Yes |
| Wi-Fi backup + captive portal | Yes (fallback AP + manual SSID) | Open `dongle-*` AP |
| Zigbee TCP serial bridge | Port **6638**, 115200 8N1, no RTS/CTS | Same |
| Home Assistant native API | Yes | No (typically) |
| Bluetooth Proxy (HA) | Yes | No |
| RGB status LED (GPIO 4/14/2) | One `Status LED` light, controllable in HA / web UI | Three fixed-function LEDs |
| Web OTA for **ESP32** | Yes (`.ota.bin` on device page) | Stock OTA |
| Web flash for **CC2652** coordinator | **Not included** | Stock tooling |
| Zigbee Reset / Enter Bootloader buttons | Yes (web UI + HA) | — |

---

## Differences vs ETH-MG optional firmware

| Item | ETH-52P (CC2652) | ETH-MG (EFR32) |
|------|------------------|----------------|
| Z2M adapter | **`zstack`** | `ember` |
| ZHA radio type | **ZNP** | EZSP |
| BSL pin | **GPIO12**, idle **high**; bootloader = BSL **low** while nRESET is released | GPIO33, idle low; bootloader = BSL high |
| Chip identification | Fixed text `CC2652P2 / CC2652P7` | GPIO35 strap (MG21 / MG24) |
| Fallback AP | `Inswift-ETH-52P` / `inswift123` | `Inswift-ETH-MG` / `inswift123` |
| Status LED | RGB on GPIO 4 / 14 / 2 | Same |

---

## Quick start — flash the ESP32

### A. First install (USB / factory image)

1. Put the gateway in download mode if required by your board (hold BOOT / use the factory cable as in the product manual).
2. Flash `firmware/inswift-eth-52p-esphome-0.1.1.factory.bin` with [esptool](https://docs.espressif.com/projects/esptool/) or the ESPHome web flasher:

   ```bash
   esptool.py --chip esp32 write_flash 0x0 inswift-eth-52p-esphome-0.1.1.factory.bin
   ```

3. Connect Ethernet (recommended) or wait ~1 minute for the fallback Wi-Fi AP `Inswift-ETH-52P` / password `inswift123`.
4. Open `http://<device-ip>/` (or the captive portal). The mDNS name is `dongle-<mac>.local`.

### B. Upgrade over the network (OTA)

1. On the device web page, use **OTA Update** at the bottom.
2. Upload `firmware/inswift-eth-52p-esphome-0.1.1.ota.bin` (ESP32 only — **not** a CC2652 coordinator image).
3. Wait for reboot. Zigbee TCP port 6638 comes back after boot.

### C. Build / adopt in Home Assistant ESPHome add-on

1. Copy `inswift-eth-52p.yaml` and the `components/` folder into `/config/esphome/` (or your ESPHome config directory).
2. Install ESPHome **2026.9.1** (or compatible).
3. Validate and install from the add-on, or adopt the already-flashed device via the ESPHome / HA API.

See [OPERATION_GUIDE.md](OPERATION_GUIDE.md) for a full walkthrough.

---

## Connect Zigbee2MQTT

Point Z2M at the TCP bridge (same as stock):

```yaml
serial:
  port: tcp://192.168.x.x:6638
  adapter: zstack
  baudrate: 115200
  rtscts: false
```

Replace the IP with the gateway’s Ethernet (or Wi-Fi) address. The CC2652 must run a **Z-Stack coordinator** firmware (115200, no flow control).

Stop Z2M before using **Zigbee Enter Bootloader** or an external flasher on port 6638, or the client will fight for the socket.

---

## Connect ZHA (Home Assistant)

Add a Zigbee Home Automation integration with a **network** / socket serial port:

- Port: `socket://192.168.x.x:6638`
- Radio type: **ZNP** (Texas Instruments Z-Stack)

Exact UI labels vary by HA version; the important part is the socket URL and the ZNP radio type.

---

## Wi-Fi setup

Ethernet is preferred (`network.priority: [ethernet, wifi]`).

**Optional Wi-Fi (also works while Ethernet is up):**

1. On the device web UI (or HA entities), set **WiFi SSID Input** and **WiFi Password Input**.
2. Press **WiFi Connect**.
3. Or call the HA action `esphome.<device>_set_wifi` with `ssid` / `password`.

**Fallback AP (defaults):** if neither Ethernet nor saved Wi-Fi is available for ~1 minute, SSID `Inswift-ETH-52P` / password `inswift123` starts with captive portal. Advanced users can change these in the YAML `wifi.ap` section.

---

## Buttons (web UI / Home Assistant)

| Button | Effect |
|--------|--------|
| **WiFi Connect** | Apply SSID/password from the text inputs |
| **Restart ESP32** | Soft reboot |
| **Factory Reset ESP32** | Clear ESPHome Wi-Fi / NVS (does not touch the CC2652 flash) |
| **Zigbee Reset** | BSL held high, nRESET low 50 ms → normal restart of the CC2652 |
| **Zigbee Enter Bootloader** | BSL low while nRESET is released → CC2652 ROM bootloader on TCP 6638 (for coordinator flashing; not yet verified on this build) |

Physical **Button** on GPIO34 is exposed as a binary sensor (active low).

---

## Pin map (ESP32)

| GPIO | Function | Notes |
|------|----------|-------|
| 0 | ETH clock in | `CLK_EXT_IN` |
| 2 | LED blue | LEDC / RGB (strapping pin) |
| 4 | LED red | LEDC / RGB |
| 5 | ETH PHY power | |
| 12 | CC2652 BSL | Idle **high**; **low** during reset release = ROM bootloader (strapping pin) |
| 13 | UART2 RX | ← CC2652 TX |
| 14 | LED green | LEDC / RGB |
| 15 | CC2652 nRESET | Idle **high** |
| 17 | UART2 TX | → CC2652 RX |
| 18 | ETH MDIO | |
| 23 | ETH MDC | |
| 34 | User button | Input-only, inverted, external pull-up |

Ethernet PHY: ESPHome `type: IP101`, `phy_addr: 1` (same as Tasmota `EthType 1`).

For a Tasmota GPIO backlog and TCP bridge rules, see [tasmota/eth-52p/README.md](../../tasmota/eth-52p/README.md).

---

## Known limitations

- **No automatic P2 / P7 detection.** The Zigbee Chip sensor is fixed text; check Z2M / ZHA for the real chip.
- **No in-browser CC2652 flasher.** To update the coordinator, stop Z2M/ZHA, press **Zigbee Enter Bootloader**, then use a PC tool such as `cc2538-bsl` over `socket://IP:6638`. This path is provided but has **not yet been verified** with 0.1.1. Coordinator images are in [ti-cc2652/](../../ti-cc2652/README.md) — pick the P2 or P7 image that matches your unit.
- The stock short-press / long-press button functions (LED mute, USB/network mode switch, hold-to-reset) are not reimplemented. Use the **Factory Reset ESP32** button instead.
- USB logger is enabled (`baud_rate: 115200`, `level: INFO`). Set `baud_rate: 0` in the YAML and rebuild for a quiet UART.
- This firmware replaces the stock ESP32 application. Keep a path to reflash stock or this package’s factory image.

---

## Recovery tips

1. **Z2M/ZHA disconnects after ESP32 OTA** — wait for reboot; confirm port 6638 with `nc <IP> 6638` or restart the integration.
2. **Coordinator stuck / not responding** — press **Zigbee Reset**; ensure no second client holds 6638; confirm 115200 / no flow control.
3. **Coordinator left in bootloader** — press **Zigbee Reset** (or power-cycle) to return to normal Z-Stack run.
4. **Ethernet no link** — check cable and PHY power (GPIO5); try Wi-Fi / AP as fallback.
5. **Broken ESP32 app** — reflash `.factory.bin` over USB with esptool.

---

## Package contents (this repo)

```
esphome/eth-52p/
  inswift-eth-52p.yaml
  components/stream_server/   # local stream_server for TCP 6638
  README.md
  OPERATION_GUIDE.md
  firmware/
    inswift-eth-52p-esphome-0.1.1.factory.bin
    inswift-eth-52p-esphome-0.1.1.ota.bin
```

Tasmota notes live under `tasmota/eth-52p/`. Overview: [docs/optional-firmware.md](../../docs/optional-firmware.md).

---

## License / upstream

- Device hardware and stock firmware: Inswift.
- ESPHome config and docs in this package: provided for community use with the hardware.
- `stream_server` is based on [oxan/esphome-stream-server](https://github.com/oxan/esphome-stream-server), adapted for ESPHome 2026.9 (same component as `esphome/eth-mg/`).

Product / docs site: [inswift.tech](https://inswift.tech) · Hardware repo: [Inswift-dev/inswift-eth](https://github.com/Inswift-dev/inswift-eth)
