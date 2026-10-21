# Inswift ETH-52P — Optional Tasmota Configuration

Community notes for running **Tasmota** on ETH-52P (CC2652P2) / ETH-52P7 (CC2652P7) as an Ethernet Zigbee TCP bridge.  
Stock firmware remains the supported default.

> Status: verified on ETH-52P hardware on 2026-10-21 — **Tasmota32 Sonoff-ZigbeeBridgePro** with the backlog below, Ethernet + TCP bridge on port 6638, and both **Zigbee2MQTT** (`zstack`) and **ZHA** (ZNP) connected successfully. Not verified: flashing the CC2652 from Tasmota or using **Zigbee Rst 2** for bootloader entry (see below).

GPIO codes below follow Tasmota development `tasmota_template.h` (**code = index × 32 + subtype**).

---

## Firmware build

On [Tasmota web install](https://tasmota.github.io/install/) choose:

**Tasmota32 Sonoff-ZigbeeBridgePro** (`tasmota32-zbbrdgpro` / `FIRMWARE_ZBBRDGPRO`)

Reasons:

- Standard Tasmota32 builds often **omit** the TCP serial bridge commands.
- ZigbeeBridgePro includes **Ethernet** support and **TCPStart** / TCP bridge used like Sonoff ZBBridge Pro.
- The Zigbee stack for Z2M/ZHA still runs on the **CC2652** (Z-Stack coordinator); the ESP32 only bridges UART ↔ TCP.

---

## One-time console backlog

Paste into the Tasmota console:

```
Backlog Module 0; GPIO0 0; GPIO5 5536; GPIO23 5568; GPIO18 5600; GPIO17 5472; GPIO13 5504; GPIO15 5792; GPIO12 3840; GPIO34 64; GPIO4 416; GPIO14 417; GPIO2 418; EthType 1; EthAddress 1; EthClockMode 0; Ethernet 1
```

After reboot:

```
Backlog TCPBaudRate 115200; TCPConfig 8N1; Rule1 ON System#Boot DO TCPStart 6638 ENDON; Rule1 1; Restart 1
```

### Template (same mapping)

```json
{"NAME":"Inswift ETH-52P","ARCH":"ESP32","GPIO":[0,0,418,0,416,5536,0,0,3840,5504,417,5792,0,5472,5600,0,0,0,0,5568,0,0,0,0,0,0,0,0,0,0,64,0,0,0,0,0],"FLAG":0,"BASE":1,"CMND":"EthType 1|EthAddress 1|EthClockMode 0|Ethernet 1"}
```

Tasmota ESP32 templates have 36 entries in this GPIO order: `0-5, 9, 10, 12-27, 6, 7, 8, 11, 32-39` (so array index 8 = GPIO12, index 30 = GPIO34). The TCP bridge rule above is still needed after applying the template.

---

## Connect Zigbee2MQTT / ZHA

Zigbee2MQTT:

```yaml
serial:
  port: tcp://192.168.x.x:6638
  adapter: zstack
  baudrate: 115200
  rtscts: false
```

ZHA: port `socket://192.168.x.x:6638`, radio type **ZNP**.

Port **6638** matches the stock Inswift bridge.

---

## Pin map

| GPIO | Tasmota function | Code | Notes |
|------|------------------|------|-------|
| 0 | None (ETH clock in) | 0 | Owned by `EthClockMode 0` |
| 5 | ETH POWER | 5536 | PHY power |
| 23 | ETH MDC | 5568 | |
| 18 | ETH MDIO | 5600 | |
| 17 | TCP Tx | 5472 | ESP TX → CC2652 RX |
| 13 | TCP Rx | 5504 | ESP RX ← CC2652 TX |
| 15 | Zigbee Rst 1 | 5792 | nRESET, idle high |
| 12 | **Output Hi** | **3840** | Hold BSL **high** for normal run — see note below |
| 34 | Button_n 1 | 64 | Active low, no internal pull-up |
| 4 | PWM 1 | 416 | RGB red |
| 14 | PWM 2 | 417 | RGB green |
| 2 | PWM 3 | 418 | RGB blue |

Ethernet: `EthType 1` (ESPHome `IP101`), `EthAddress 1`, `EthClockMode 0` (GPIO0_IN).

---

## Note: GPIO12 / BSL polarity (CC2652)

On ETH-52P the CC2652 BSL pin is **idle high**; the ROM bootloader is entered when BSL is **low** while nRESET is released. This is the **opposite** of Inswift ETH-MG (EFR32), where BSL is held low.

- Use **Output Hi (3840)** on GPIO12 for daily use (this is the verified configuration).
- Do **not** copy the ETH-MG mapping (**Output Lo** on the BSL pin) — holding BSL low can leave the CC2652 in its bootloader after a reset.
- Tasmota's **Zigbee Rst 2** (5793) is documented as an active-low bootloader pin and may suit this board, but it has **not been verified** on ETH-52P. Flashing the CC2652 through Tasmota is likewise unverified. To update the coordinator, use stock tooling (the ESPHome optional firmware's **Zigbee Enter Bootloader** path is also not yet verified).

---

## Other notes

- Stock CC2652 coordinator firmware: **115200**, **no flow control** — match `TCPBaudRate` / `TCPConfig`.
- The three LEDs are driven as one RGB light; set it from the UI or with `Color`, e.g. `Color 00FF00` for green.
- The stock short/long-press button functions are not available under Tasmota.
- Flashing Tasmota replaces the stock ESP32 web UI and OTA path — keep a way to restore stock or the ESPHome factory image.
- Prefer the TCP bridge with external Z2M/ZHA. Zigbee2Tasmota on this board has not been tested.

---

## Related

- ESPHome optional firmware: [../../esphome/eth-52p/README.md](../../esphome/eth-52p/README.md)  
- Operation guide: [../../esphome/eth-52p/OPERATION_GUIDE.md](../../esphome/eth-52p/OPERATION_GUIDE.md)  
- Optional firmware hub: [../../docs/optional-firmware.md](../../docs/optional-firmware.md)  
