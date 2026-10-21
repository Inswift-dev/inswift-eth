# Prebuilt ESPHome images — Inswift ETH-52P 0.1.1

These binaries match the optional ESPHome package in the parent directory (`inswift-eth-52p.yaml`). One image covers both ETH-52P (CC2652P2) and ETH-52P7 (CC2652P7).

| File | Use |
|------|-----|
| `inswift-eth-52p-esphome-0.1.1.factory.bin` | First install over USB (`esptool write_flash 0x0 …`) |
| `inswift-eth-52p-esphome-0.1.1.ota.bin` | ESP32 OTA from the device web page (not for CC2652 coordinator images) |

See [../README.md](../README.md) and [../OPERATION_GUIDE.md](../OPERATION_GUIDE.md) for flashing and operation. Stock firmware remains the default; see [../../../docs/optional-firmware.md](../../../docs/optional-firmware.md).
