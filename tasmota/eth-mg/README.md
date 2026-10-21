# Inswift ETH-MG — Optional Tasmota Configuration

Draft / community notes for running **Tasmota** on ETH-MG21 / ETH-MG24 (including PoE).  
Stock firmware remains the supported default. Validate on your unit before production use.

GPIO codes below follow Tasmota development `tasmota_template.h` (**code = index × 32 + subtype**).

---

## Firmware build

On [Tasmota web install](https://tasmota.github.io/install/) choose:

**Tasmota32 Sonoff-ZigbeeBridgePro** (`FIRMWARE_ZBBRDGPRO`)

Reasons:

- Standard Tasmota32 builds often **omit** the TCP serial bridge commands.
- ZigbeeBridgePro includes **Ethernet** support and **TCPStart** / TCP bridge used like Sonoff ZBBridge Pro.
- Zigbee stack for Z2M/ZHA still runs on the **EFR**; ESP only bridges UART ↔ TCP.

Z2M/ZHA keep using `tcp://IP:6638` (or `socket://IP:6638` for ZHA).

---

## One-time console backlog

Paste into the Tasmota console:

```
Backlog Module 0; GPIO0 0; GPIO5 5536; GPIO23 5568; GPIO18 5600; GPIO17 5472; GPIO13 5504; GPIO15 5792; GPIO33 3872; GPIO34 64; GPIO4 416; GPIO14 417; GPIO2 418; EthType 1; EthAddress 1; EthClockMode 0; Ethernet 1
```

After reboot:

```
Backlog TCPBaudRate 115200; TCPConfig 8N1; Rule1 ON System#Boot DO TCPStart 6638 ENDON; Rule1 1; Restart 1
```

---

## Pin map

| GPIO | Tasmota function | Code | Notes |
|------|------------------|------|-------|
| 0 | None (ETH clock in) | 0 | Owned by `EthClockMode 0` |
| 5 | ETH POWER | 5536 | PHY power |
| 23 | ETH MDC | 5568 | |
| 18 | ETH MDIO | 5600 | |
| 17 | TCP Tx | 5472 | ESP TX → EFR RX |
| 13 | TCP Rx | 5504 | ESP RX ← EFR TX |
| 15 | Zigbee Rst 1 | 5792 | nRESET, idle high |
| 33 | **Output Lo** | **3872** | Hold BSL **low** — see warning below |
| 34 | Button_n 1 | 64 | Active low, no internal pull-up |
| 4 | PWM 1 | 416 | RGB red |
| 14 | PWM 2 | 417 | RGB green |
| 2 | PWM 3 | 418 | RGB blue |
| 35 | Unassigned | — | MG21/MG24 detect; unused by Tasmota |

Ethernet: `EthType 1` (ESPHome `IP101`), `EthAddress 1`, `EthClockMode 0` (GPIO0_IN).

---

## Critical warning: GPIO33 / BSL polarity

**Do not** map GPIO33 as Tasmota **Zigbee Rst 2**.

Tasmota’s Zigbee Rst 2 behaviour follows Sonoff ZBBridge Pro BOOT polarity. On Inswift ETH-MG, **BSL must be high to enter the Gecko bootloader** and **low for normal NCP run**. Using Zigbee Rst 2 can leave the EFR stuck in bootloader (or otherwise mis-driven).

Safe approach for daily use: **Output Lo (3872)** so BSL stays low. Enter bootloader with a careful temporary high drive only when you intend to flash, or use ESPHome / external tools with the documented pulse sequence.

---

## Other notes

- Stock EFR coordinator firmware: **115200**, **no flow control** — match `TCPBaudRate` / `TCPConfig`.
- RGB LEDs can be set from the UI or with `Color`, e.g. `Color 00FF00` for green.
- Flashing Tasmota replaces the stock ESP32 web UI and OTA path — keep a way to restore stock or ESPHome factory images.
- Alternative: Zigbee2Tasmota (GPIO17 = Zigbee Tx **3552**, GPIO13 = Zigbee Rx **3584**) only if you confirm EmberZNet NCP version support; TCP bridge + external Z2M/ZHA is the path validated with stock NCP usage.

---

## Related

- ESPHome optional firmware: [../../esphome/eth-mg/README.md](../../esphome/eth-mg/README.md)  
- Operation guide: [../../esphome/eth-mg/OPERATION_GUIDE.md](../../esphome/eth-mg/OPERATION_GUIDE.md)  
- Optional firmware hub: [../../docs/optional-firmware.md](../../docs/optional-firmware.md)  
