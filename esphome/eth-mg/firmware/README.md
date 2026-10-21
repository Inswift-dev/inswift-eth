# Prebuilt ESPHome images — Inswift ETH-MG 0.1.8

These binaries match the optional ESPHome package in the parent directory (`inswift-eth-mg.yaml`).

| File | Use |
|------|-----|
| `inswift-eth-mg-esphome-0.1.8.factory.bin` | First install over USB (`esptool write_flash 0x0 …`) |
| `inswift-eth-mg-esphome-0.1.8.ota.bin` | ESP32 OTA from the device web page (not for Zigbee `.gbl`) |

See [../README.md](../README.md) and [../OPERATION_GUIDE.md](../OPERATION_GUIDE.md) for flashing and operation. Stock firmware remains the default; see [../../../docs/optional-firmware.md](../../../docs/optional-firmware.md).
