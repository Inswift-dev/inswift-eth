# ZBM-MG dongle IO configuration

> Source: Inswift website tutorial [ZBM-MG dongle IO configuration](https://inswift.tech/en/tutorials/zbm-mg-dongle-io/) ([中文](https://inswift.tech/tutorials/zbm-mg-dongle-io/)). If anything here differs from the website, the website is authoritative.

For stock images, use [Flash USB dongle firmware](https://inswift.tech/en/tutorials/flash-zbm-mg24/). You do not need these pins.

The I/O below is for custom firmware: building your own coordinator / router / Thread image, porting another project, or writing a flasher. Match UART, BOOT, reset, and how the USB-serial chip drives auto-download.

This page covers **ZBM-MG21 / ZBM-MG24**. ZBT-1P follows the official SkyConnect pinout (not listed here). Its USB chip is in the last table for comparison.

## Pins

Both boards wire USART to USB-serial without hardware flow control. BOOT is active-low. NRST is driven by the DTR / RTS circuit, so you do not press reset by hand while flashing.

| Item | ZBM-MG21 | ZBM-MG24 |
| --- | --- | --- |
| Silicon | EFR32MG21A020F768IM32 | EFR32MG24A420F1536IM40 |
| USART TX | PB01 | PC01 |
| USART RX | PB00 | PC02 |
| Flow control | None | None |
| BOOT | PA00 (active-low) | PC00 (active-low) |
| RESET | NRST, via DTR / RTS | NRST, via DTR / RTS |
| LED | None | None |
| Baud (recommended) | 115200 | 115200 |

When you bring up custom firmware:

- Do not swap TX / RX, and do not move USART to other pins unless the board wiring changes too
- Do not enable RTS / CTS on this UART; those lines are not wired to the radio
- Prefer 115200. If you change baud, match the flasher and the ZHA / Zigbee2MQTT serial settings

## Auto-download

The USB-serial chip uses DTR and RTS to drive NRST and BOOT, so the web flasher and the Windows tool can enter boot without shorting the BOOT pin.

Truth table (`1` / `0` are DTR and RTS levels):

| DTR | RTS | NRST | BOOT |
| --- | --- | --- | --- |
| 1 | 1 | released | released |
| 0 | 0 | released | released |
| 1 | 0 | reset | released |
| 0 | 1 | released | enter boot |

Drive DTR / RTS this way in your own downloader: enter boot, then send the image over USART. Do not toggle these lines during normal serial traffic, or the stick will reset or sit in boot.

## USB serial

Driver links are in [Flash USB dongle firmware](https://inswift.tech/en/tutorials/flash-zbm-mg24/). Custom scripts can key off VID:PID:

| Product | USB chip | VID:PID | Typical node |
| --- | --- | --- | --- |
| ZBM-MG21 | WCH CH343 / CH9102 | `1a86:55d4` | `/dev/ttyACM0` |
| ZBM-MG24 | Silicon Labs CP210x | `10c4:ea60` | `/dev/ttyUSB0` |
| ZBT-1P | CP2102N (same as official ZBT-1) | `10c4:ea60` | `/dev/ttyUSB*` |

ZBM-MG24 and ZBT-1P share a VID:PID. Do not tell them apart by USB ID alone; use the enclosure or the model the user selected.

To flash official firmware only, use the [USB dongle model flasher](https://inswift.tech/en/tools/usb-dongle/) or [Inswift Silabs Flasher](https://github.com/Inswift-dev/inswift-silabs-flasher). You do not need to drive these I/O lines yourself.
