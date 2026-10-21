# Operation Guide — Inswift ETH-MG ESPHome 0.1.8

Step-by-step install and day-to-day use for the optional ESPHome firmware.

---

## 1. What you need

- Inswift ETH-MG21, ETH-MG24, or PoE variant
- Ethernet cable (recommended) and DHCP network
- Either:
  - USB serial access for the first flash, **or**
  - Ability to install a `.factory.bin` / use an existing ESPHome OTA path
- Zigbee2MQTT or ZHA if you use the gateway as a coordinator
- Matching **EmberZNet NCP** `.gbl` already on the EFR (stock NCP is fine; 115200, no flow control)

---

## 2. Install options

### 2.1 Standalone: flash prebuilt binaries

1. Use the images under `esphome/eth-mg/firmware/` in this repository (or download them from the release assets on GitHub).
2. Flash the factory image (USB):

   ```bash
   esptool.py --chip esp32 -p /dev/ttyUSB0 write_flash 0x0 inswift-eth-mg-esphome-0.1.8.factory.bin
   ```

   On Windows, replace the port with `COMx`.

3. Power the device, plug Ethernet, wait for DHCP.
4. Find the IP (router DHCP list, or mDNS `inswift-eth-mg-XXXXXX.local`).
5. Open `http://<IP>/` in a browser.

### 2.2 Home Assistant — ESPHome add-on (build from YAML)

1. Install the **ESPHome** add-on (or ESPHome Device Builder) if you do not have it.
2. Copy into the ESPHome config directory (often `/config/esphome/` on HA OS):

   - `inswift-eth-mg.yaml`
   - entire `components/` directory (must include `components/stream_server/`)

3. Ensure ESPHome version is **2026.9.x** (2026.9.1 tested).
4. In the ESPHome UI: open the config → **Install** → choose USB, or OTA if the device is already running ESPHome.
5. After first boot, **Adopt** / add the device via the ESPHome integration (native API). Encryption keys are prompted on first connect if you enable API encryption later.

### 2.3 Upgrade an existing 0.1.x ESPHome install

1. Open the device web page → **OTA Update**.
2. Upload `inswift-eth-mg-esphome-0.1.8.ota.bin`.
3. Do **not** upload a Zigbee `.gbl` here — that path is ESP32-only.

---

## 3. First boot checklist

1. **Ethernet:** Status LED defaults to green. Confirm **Ethernet IP** on the web page or HA.
2. **No cable:** After about one minute, join the fallback Wi-Fi AP (SSID `Inswift-ETH-MG`, password `inswift123`), open the captive portal, and set home Wi-Fi — or plug Ethernet instead.
3. **Zigbee Chip** text sensor should read `EFR32MG21` or `EFR32MG24` (from GPIO35).
4. From a PC: `nc <IP> 6638` or start Z2M — the TCP bridge should accept a client.

---

## 4. Networking behaviour

| Situation | Behaviour |
|-----------|-----------|
| Ethernet link up | Prefer Ethernet for HA API, web UI, Zigbee TCP |
| Ethernet down, Wi-Fi configured | Use Wi-Fi |
| Neither up for ~1 min | Fallback AP + captive portal |
| Wi-Fi down while Ethernet up | No reboot (`reboot_timeout: 0s`) |

To add Wi-Fi while Ethernet is already working: fill **WiFi SSID Input** / **WiFi Password Input**, press **WiFi Connect**.

Fallback AP defaults for this firmware: SSID `Inswift-ETH-MG`, password `inswift123`. Advanced users can change them under `wifi.ap` in the YAML.

---

## 5. Connect Zigbee2MQTT

Example `configuration.yaml` fragment:

```yaml
serial:
  port: tcp://192.168.100.105:6638
  adapter: ember
  baudrate: 115200
  rtscts: false
```

1. Stop any other client using 6638.
2. Set the IP to your gateway.
3. Start Z2M; the coordinator should interview as usual if the EFR NCP firmware is healthy.

If you see ASH / `HOST_FATAL_ERROR` loops: press **Zigbee Reset**, confirm baud/flow-control, and ensure only one TCP client is connected.

---

## 6. Connect ZHA

1. Settings → Devices & services → Add integration → **Zigbee Home Automation**.
2. Choose serial / radio setup that allows a **socket** path.
3. Port: `socket://192.168.x.x:6638`
4. Radio: **EZSP**

Do not run ZHA and Z2M against the same gateway at the same time.

---

## 7. LED and button meanings

### Status LED (RGB)

| Colour / effect | Typical meaning |
|-----------------|-----------------|
| Green (default first boot) | Firmware running |
| User-set colour / pulse / strobe | Controlled from HA or web UI (`Status LED` light entity) |

GPIO map: red = GPIO4, green = GPIO14, blue = GPIO2 (active via LEDC).

### Soft buttons

| Name | When to use |
|------|-------------|
| Zigbee Reset | Soft-restart the EFR without entering bootloader |
| Zigbee Enter Bootloader | Before PC-side `.gbl` flash via TCP 6638 |
| WiFi Connect | Apply new SSID/password |
| Restart ESP32 | Reboot host MCU |
| Factory Reset ESP32 | Clear saved Wi-Fi / ESPHome NVS |

### Physical button

GPIO34, active low — exposed as **Button** binary sensor. Wire your own automations in HA if needed.

---

## 8. Updating Zigbee (EFR) firmware

Web UI OTA on this build updates **ESP32 only**.

To update the EFR:

1. Stop Z2M / ZHA.
2. Press **Zigbee Enter Bootloader** on the device page.
3. Immediately run a flasher, for example:

   ```bash
   universal-silabs-flasher --device socket://192.168.x.x:6638 flash --firmware your-ncp.gbl
   ```

4. Use an NCP image matching **MG21 vs MG24**. RCP / OpenThread images will not work with Z2M/ZHA ember.
5. Press **Zigbee Reset** (or power-cycle) if needed, then start Z2M/ZHA again.

A future firmware may add in-browser `.gbl` flashing; it is **not** part of 0.1.8.

---

## 9. Troubleshooting

| Symptom | What to try |
|---------|-------------|
| No web UI | Check Ethernet IP; try AP `Inswift-ETH-MG`; reflash factory bin |
| Z2M cannot open port | Confirm `tcp://IP:6638`; only one client; ESP finished booting |
| ASH / HOST_FATAL_ERROR | Zigbee Reset; correct NCP firmware; baud 115200; no RTS/CTS |
| Ethernet never links | Cable/switch; GPIO5 PHY power; use Wi-Fi interim |
| Wrong chip label | GPIO35 sense — compare with board SKU |
| After ESP OTA, Zigbee gone | Wait for full boot; restart Z2M; check 6638 |
| Want USB logs | Set `logger.baud_rate: 115200`, `level: DEBUG`, rebuild |

---

## 10. Related docs

- [README.md](README.md) — overview, pin map, package list  
- [../../tasmota/eth-mg/README.md](../../tasmota/eth-mg/README.md) — optional Tasmota ZigbeeBridgePro template  
- [../../docs/optional-firmware.md](../../docs/optional-firmware.md) — optional firmware hub  
