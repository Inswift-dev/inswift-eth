# Inswift ETH-MG — Optional ESPHome Firmware

**Version:** 0.1.8  
**Target:** ESPHome 2026.9.1 (esp-idf)  
**Products:** ETH-MG21 · ETH-MG24 · ETH-MG21 PoE · ETH-MG24 PoE

This package is an **optional advanced firmware** for Inswift ETH-MG Ethernet Zigbee gateways.  
**Stock (official) firmware remains the default** and is fully supported. Use this build if you want ESPHome integration, Home Assistant Bluetooth Proxy, a documented pin map, and the same Zigbee TCP bridge as stock (`tcp://IP:6638`).

> Status: Ethernet + Zigbee TCP bridge + Z2M/ZHA were verified on hardware with the 0.1.7-debug lineage. 0.1.8 is the same feature set without UART debug noise (release logger settings).

---

## Supported models

| Model | Zigbee SoC | How the firmware detects it |
|-------|------------|-----------------------------|
| ETH-MG21 / ETH-MG21 PoE | EFR32MG21 (768 KB / 64 KB) | GPIO35 **high** |
| ETH-MG24 / ETH-MG24 PoE | EFR32MG24 (1536 KB / 256 KB) | GPIO35 **low** |

All four SKUs share the same ESP32 pinout. PoE units only differ in power delivery; this firmware treats them the same.

---

## Features (vs stock)

| Feature | This firmware | Stock |
|---------|---------------|-------|
| Ethernet primary networking | Yes (IP101 / EthType 1) | Yes |
| Wi-Fi backup + captive portal | Yes (fallback AP + manual SSID) | Varies |
| Zigbee TCP serial bridge | Port **6638**, 115200 8N1, no RTS/CTS | Same |
| Home Assistant native API | Yes | No (typically) |
| Bluetooth Proxy (HA) | Yes | No |
| RGB status LED (GPIO 4/14/2) | Controllable in HA / web UI | Fixed behaviour |
| Web OTA for **ESP32** | Yes (`.ota.bin` on device page) | Stock OTA |
| Web flash for **EFR / Zigbee** `.gbl` | **Not included yet** (deferred) | Use PC tools / stock path |
| Zigbee Reset / Enter Bootloader buttons | Yes (web UI + HA) | — |

---

## Quick start — flash the ESP32

### A. First install (USB / factory image)

1. Put the gateway in download mode if required by your board (hold BOOT / use the factory cable as in the product manual).
2. Flash `inswift-eth-mg-esphome-0.1.8.factory.bin` with [esptool](https://docs.espressif.com/projects/esptool/) or the ESPHome web flasher:

   ```bash
   esptool.py --chip esp32 write_flash 0x0 inswift-eth-mg-esphome-0.1.8.factory.bin
   ```

3. Connect Ethernet (recommended) or wait ~1 minute for the fallback Wi-Fi AP `Inswift-ETH-MG` / password `inswift123`.
4. Open `http://<device-ip>/` (or the captive portal). Note the Ethernet IP shown on the page / in HA.

### B. Upgrade over the network (OTA)

1. On the device web page, use **OTA Update** at the bottom.
2. Upload `inswift-eth-mg-esphome-0.1.8.ota.bin` (ESP32 only — **not** a Zigbee `.gbl`).
3. Wait for reboot. Zigbee TCP port 6638 comes back after boot.

### C. Build / adopt in Home Assistant ESPHome add-on

1. Copy `inswift-eth-mg.yaml` and the `components/` folder into `/config/esphome/` (or your ESPHome config directory).
2. Install ESPHome **2026.9.1** (or compatible).
3. Validate and install from the add-on, or adopt the already-flashed device via the ESPHome / HA API.

See [OPERATION_GUIDE.md](OPERATION_GUIDE.md) for a full walkthrough.

---

## Connect Zigbee2MQTT

Point Z2M at the TCP bridge (same as stock):

```yaml
serial:
  port: tcp://192.168.x.x:6638
  adapter: ember
  baudrate: 115200
  rtscts: false
```

Replace the IP with the gateway’s Ethernet (or Wi-Fi) address. Use an **EmberZNet NCP** firmware on the EFR that matches your chip (MG21 vs MG24), 115200, no flow control.

Stop Z2M before using **Zigbee Enter Bootloader** or external flashers on port 6638, or the client will fight for the socket.

---

## Connect ZHA (Home Assistant)

Add a Zigbee Home Automation integration with a **network** / socket serial port:

- Port: `socket://192.168.x.x:6638`
- Radio type: **EZSP** (EmberZNet)

Exact UI labels vary by HA version; the important part is the socket URL and EZSP/ember radio.

---

## Wi-Fi setup

Ethernet is preferred (`network.priority: [ethernet, wifi]`).

**Optional Wi-Fi (also works while Ethernet is up):**

1. On the device web UI (or HA entities), set **WiFi SSID Input** and **WiFi Password Input**.
2. Press **WiFi Connect**.
3. Or call the HA action `esphome.<device>_set_wifi` with `ssid` / `password`.

**Fallback AP (defaults):** if neither Ethernet nor saved Wi-Fi is available for ~1 minute, SSID `Inswift-ETH-MG` / password `inswift123` starts with captive portal. Advanced users can change these in the YAML `wifi.ap` section.

---

## Buttons (web UI / Home Assistant)

| Button | Effect |
|--------|--------|
| **WiFi Connect** | Apply SSID/password from the text inputs |
| **Restart ESP32** | Soft reboot |
| **Factory Reset ESP32** | Clear ESPHome Wi-Fi / NVS (does not rewrite EFR flash) |
| **Zigbee Reset** | Pulse EFR nRESET with BSL low (normal restart of Zigbee SoC) |
| **Zigbee Enter Bootloader** | Pulse reset with BSL high so Gecko bootloader can be reached on TCP 6638 |

Physical **Button** on GPIO34 is exposed as a binary sensor (active low).

---

## Pin map (ESP32)

| GPIO | Function | Notes |
|------|----------|-------|
| 0 | ETH clock in | `CLK_EXT_IN` |
| 2 | LED blue | LEDC / RGB |
| 4 | LED red | LEDC / RGB |
| 5 | ETH PHY power | |
| 13 | UART2 RX | ← EFR TX |
| 14 | LED green | LEDC / RGB |
| 15 | EFR nRESET | Idle **high** |
| 17 | UART2 TX | → EFR RX |
| 18 | ETH MDIO | |
| 23 | ETH MDC | |
| 33 | EFR BSL | **High** = bootloader on reset; keep low for normal run |
| 34 | User button | Input-only, inverted, external pull-up |
| 35 | Model detect | High = MG21, low = MG24 (input-only) |

Ethernet PHY: ESPHome `type: IP101`, `phy_addr: 1` (same as Tasmota `EthType 1`).

For a Tasmota GPIO backlog and TCP bridge rules, see [tasmota/eth-mg/README.md](../../tasmota/eth-mg/README.md).

---

## Known limitations

- **No in-browser EFR / `.gbl` flasher yet.** An experimental web flasher was prototyped and deferred after bootloader entry failed on hardware. Flash Zigbee firmware with `universal-silabs-flasher` (or similar) over `socket://IP:6638` after **Zigbee Enter Bootloader**, or use stock tooling.
- Logger USB is quiet (`baud_rate: 0`). For serial debug, set `baud_rate: 115200` and `level: DEBUG` in the YAML and rebuild.
- Fallback AP defaults: SSID `Inswift-ETH-MG`, password `inswift123` (documented in YAML; advanced users may change `wifi.ap`).
- This firmware replaces the stock ESP32 application. Keep a path to reflash stock or this package’s factory image.

---

## Recovery tips

1. **Z2M/ZHA disconnects after ESP32 OTA** — wait for reboot; confirm port 6638 with `nc <IP> 6638` or restart the integration.
2. **EFR stuck / ASH errors** — press **Zigbee Reset**; ensure no second client holds 6638; confirm NCP baud 115200 / no flow control.
3. **Need Gecko bootloader** — stop Z2M/ZHA, press **Zigbee Enter Bootloader**, immediately run e.g.  
   `universal-silabs-flasher --device socket://IP:6638 probe`
4. **Ethernet no link** — check cable and PHY power (GPIO5); try Wi-Fi / AP as fallback.
5. **Brick ESP32 app** — reflash `.factory.bin` over USB with esptool.

---

## Package contents (this repo)

```
esphome/eth-mg/
  inswift-eth-mg.yaml
  components/stream_server/   # local stream_server for TCP 6638
  README.md
  OPERATION_GUIDE.md
  firmware/
    inswift-eth-mg-esphome-0.1.8.factory.bin
    inswift-eth-mg-esphome-0.1.8.ota.bin
```

Tasmota notes live under `tasmota/eth-mg/`. Overview: [docs/optional-firmware.md](../../docs/optional-firmware.md).

---

## License / upstream

- Device hardware and stock firmware: Inswift.
- ESPHome config and docs in this package: provided for community use with the hardware.
- `stream_server` is based on [oxan/esphome-stream-server](https://github.com/oxan/esphome-stream-server), adapted for ESPHome 2026.9.

Product / docs site: [inswift.tech](https://inswift.tech) · Hardware repo: [Inswift-dev/inswift-eth](https://github.com/Inswift-dev/inswift-eth)
