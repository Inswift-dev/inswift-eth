# Operation Guide — Inswift ETH-52P ESPHome 0.1.1

Step-by-step install and day-to-day use for the optional ESPHome firmware.

> Status: verified on hardware on 2026-10-21 with Home Assistant ZHA, Zigbee2MQTT and the ESPHome integration. Coordinator flashing via **Zigbee Enter Bootloader** (section 8) is provided but not yet verified on this build.

---

## 1. What you need

- Inswift ETH-52P (CC2652P2) or ETH-52P7 (CC2652P7)
- Ethernet cable (recommended) and DHCP network
- Either:
  - USB serial access for the first flash, **or**
  - Ability to install a `.factory.bin` / use an existing ESPHome OTA path
- Zigbee2MQTT (`adapter: zstack`) or ZHA (radio **ZNP**) if you use the gateway as a coordinator
- **Z-Stack coordinator** firmware already on the CC2652 (stock coordinator firmware is fine; 115200 8N1, no flow control)

---

## 2. Install options

### 2.1 Standalone: flash prebuilt binaries

1. Use the images under `esphome/eth-52p/firmware/` in this repository.
2. Flash the factory image (USB):

   ```bash
   esptool.py --chip esp32 -p /dev/ttyUSB0 write_flash 0x0 inswift-eth-52p-esphome-0.1.1.factory.bin
   ```

   On Windows, replace the port with `COMx`.

3. Power the device, plug Ethernet, wait for DHCP.
4. Find the IP (router DHCP list, or mDNS `dongle-XXXXXX.local`).
5. Open `http://<IP>/` in a browser.

### 2.2 Home Assistant — ESPHome add-on (build from YAML)

1. Install the **ESPHome** add-on (or ESPHome Device Builder) if you do not have it.
2. Copy into the ESPHome config directory (often `/config/esphome/` on HA OS):

   - `inswift-eth-52p.yaml`
   - entire `components/` directory (must include `components/stream_server/`)

3. Ensure ESPHome version is **2026.9.x** (2026.9.1 tested).
4. In the ESPHome UI: open the config → **Install** → choose USB, or OTA if the device is already running ESPHome.
5. After first boot, **Adopt** / add the device via the ESPHome integration (native API).

### 2.3 Upgrade an existing ESPHome install

1. Open the device web page → **OTA Update**.
2. Upload `inswift-eth-52p-esphome-0.1.1.ota.bin`.
3. Do **not** upload a CC2652 coordinator image here — that path is ESP32-only.

---

## 3. First boot checklist

1. **Ethernet:** Status LED comes up green. Confirm **Ethernet IP** on the web page or in HA.
2. **No cable:** After about one minute, join the fallback Wi-Fi AP (SSID `Inswift-ETH-52P`, password `inswift123`), open the captive portal, and set home Wi-Fi — or plug Ethernet instead.
3. **Zigbee Chip** text sensor reads `CC2652P2 / CC2652P7` (fixed text; the firmware does not probe the chip).
4. From a PC: `nc <IP> 6638` or start Z2M — the TCP bridge should accept a client.

---

## 4. Networking behaviour

| Situation | Behaviour |
|-----------|-----------|
| Ethernet link up | Prefer Ethernet for HA API, web UI, Zigbee TCP |
| Ethernet down, Wi-Fi configured | Use Wi-Fi |
| Neither up for ~1 min | Fallback AP + captive portal |
| Wi-Fi down while Ethernet up | No reboot (`reboot_timeout: 0s`) |

To add Wi-Fi while Ethernet is already working: fill **WiFi SSID Input** / **WiFi Password Input**, press **WiFi Connect** (or call the HA action `esphome.<device>_set_wifi`).

Fallback AP defaults for this firmware: SSID `Inswift-ETH-52P`, password `inswift123`. Advanced users can change them under `wifi.ap` in the YAML.

---

## 5. Connect Zigbee2MQTT

Example `configuration.yaml` fragment:

```yaml
serial:
  port: tcp://192.168.x.x:6638
  adapter: zstack
  baudrate: 115200
  rtscts: false
```

1. Stop any other client using 6638.
2. Set the IP to your gateway.
3. Start Z2M; the coordinator should start as usual if the Z-Stack firmware is healthy.

If the coordinator does not respond: press **Zigbee Reset**, confirm baud/flow-control, and ensure only one TCP client is connected.

---

## 6. Connect ZHA

1. Settings → Devices & services → Add integration → **Zigbee Home Automation**.
2. Choose serial / radio setup that allows a **socket** path.
3. Port: `socket://192.168.x.x:6638`
4. Radio: **ZNP** (Texas Instruments Z-Stack)

Do not run ZHA and Z2M against the same gateway at the same time.

---

## 7. LED and button meanings

### Status LED (RGB)

| Colour / effect | Typical meaning |
|-----------------|-----------------|
| Green (default first boot) | Firmware running |
| User-set colour / pulse / strobe | Controlled from HA or web UI (`Status LED` light entity) |

GPIO map: red = GPIO4, green = GPIO14, blue = GPIO2 (LEDC PWM). The stock firmware drives these three LEDs separately; this firmware exposes them as one RGB light, the same as the ETH-MG optional firmware.

### Soft buttons

| Name | When to use |
|------|-------------|
| Zigbee Reset | Soft-restart the CC2652 without entering the bootloader |
| Zigbee Enter Bootloader | Before PC-side coordinator flashing via TCP 6638 (see section 8) |
| WiFi Connect | Apply new SSID/password |
| Restart ESP32 | Reboot host MCU |
| Factory Reset ESP32 | Clear saved Wi-Fi / ESPHome NVS |

### Physical button

GPIO34, active low — exposed as **Button** binary sensor. The stock short/long-press functions are not reimplemented; wire your own automations in HA if needed.

---

## 8. Updating Zigbee (CC2652) firmware

Web UI OTA on this build updates **ESP32 only**.

> This path is included but **has not yet been verified** on ETH-52P with 0.1.1. If in doubt, keep using the stock firmware tooling for coordinator updates.

Reset sequences driven by the ESP32 (same timing as stock):

| Button | Sequence |
|--------|----------|
| Zigbee Reset | BSL high, nRESET low → 50 ms → nRESET high → 500 ms |
| Zigbee Enter Bootloader | BSL low + nRESET low → 50 ms → nRESET high (BSL still low) → 500 ms → BSL high → 500 ms |

The CC2652 BSL polarity is the **opposite** of the ETH-MG (EFR32) gateways.

To update the coordinator:

1. Stop Z2M / ZHA so nothing holds port 6638.
2. Press **Zigbee Enter Bootloader** on the device page.
3. Run a CC2652 flasher against the TCP socket, for example (flags vary by tool version):

   ```bash
   cc2538-bsl.py -p socket://192.168.x.x:6638 -ewv coordinator.hex
   ```

4. Use a Z-Stack **coordinator** image matching **CC2652P2 vs CC2652P7** (see [ti-cc2652/](../../ti-cc2652/README.md)). Router or Thread images will not work with Z2M `zstack` / ZHA ZNP.
5. Press **Zigbee Reset** (or power-cycle), then start Z2M/ZHA again.

---

## 9. Troubleshooting

| Symptom | What to try |
|---------|-------------|
| No web UI | Check Ethernet IP; try AP `Inswift-ETH-52P`; reflash factory bin |
| Z2M cannot open port | Confirm `tcp://IP:6638`; only one client; ESP finished booting |
| Coordinator not responding | Zigbee Reset; Z-Stack coordinator firmware; baud 115200; no RTS/CTS |
| Wrong adapter error in Z2M | Use `adapter: zstack` (not `ember`) |
| Ethernet never links | Cable/switch; GPIO5 PHY power; use Wi-Fi interim |
| Want to know P2 vs P7 | Check the coordinator info in Z2M / ZHA |
| After ESP OTA, Zigbee gone | Wait for full boot; restart Z2M; check 6638 |
| Want quieter USB serial | Set `logger.baud_rate: 0`, rebuild |

---

## 10. Related docs

- [README.md](README.md) — overview, pin map, package list  
- [../../tasmota/eth-52p/README.md](../../tasmota/eth-52p/README.md) — optional Tasmota ZigbeeBridgePro template  
- [../../docs/optional-firmware.md](../../docs/optional-firmware.md) — optional firmware hub  
- [../../ti-cc2652/README.md](../../ti-cc2652/README.md) — CC2652 coordinator images  
