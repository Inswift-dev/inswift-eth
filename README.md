**Inswift Zigbee 3.0 PoE Ethernet USB Adapter's Firmware**

**Zigbee over LAN,USB,WiFI is supported.**
<img width="2554" height="1364" alt="4" src="https://github.com/user-attachments/assets/b8ed13d2-7e4d-4bda-bda7-46fcc696703a" />

**1.SOC INFO** 

used ESP32 + TI CC2652P/P7 or Silicon Labs MG21/24

Performance ranking: EFR32MG24 > EFR32MG21 > CC652P7 > CC2652P

Compatibility ranking: CC2562P > EFR32MG24 > EFR32MG21 > CC2652P7

High performance: EFR32MG24 supports multiple protocols such as Matter and Thread

**CC2652P：** 
8MHz Cortex-M4 processor + 88KB RAM, the Zigbee firmware theoretically supports 200 Devices, with -105dBm sensitivity and +20dBm transmit power. It features balanced signal coverage and anti-interference capability, making it suitable for medium to large-scale networks. It requires configuration of "adapter: zstack", offers high cost-performance, and is fully supported by Zigbee2MQTT/ZHA.

**CC2652P7：** 
It has the same processor as the CC2652P (48MHz Cortex-M4), but the RAM is increased to 152KB, and it can theoretically handle more concurrent tasks. The firmware does not explicitly increase the device limit, but the large memory may reduce stuttering when multiple devices are running. It is similar to the CC2652P (-104dBm sensitivity), but supports advanced functions such as Mobile Target Detection (MTD). It needs to be configured with adapter: zstack, and is fully supported by Zigbee2MQTT/ZHA.

**EFR32MG21：** 
80MHz Cortex-M33 processor + 64KB RAM, with computing power superior to TI chips. However, the smaller RAM may limit its performance in complex scenarios. It features a 20dBm transmit power and -104.3dBm sensitivity, and its measured link quality (LQI) is better than that of the CC26* series. It native supports Zigbee+Bluetooth and can serve as the core of a multi-protocol gateway.

**EFR32MG24：**
78MHz Cortex-M33 processor + 256KB RAM, boasting the strongest performance. It supports AI/ML acceleration and multiple protocols (Matter/Thread), with -105.4dBm sensitivity and 19.5dBm transmit power. It delivers optimal signal coverage and anti-interference capability, and supports hardware encryption (Secure Vault) and multiple peripherals, making it suitable for enterprise-level or complex IoT applications.
<img width="773" height="535" alt="image" src="https://github.com/user-attachments/assets/7df2ee49-3107-4df1-b24a-7d636fb26d1d" />

**2.LED status**

BLUE - Working Mode zigbee usb status

GREEN - Working Mode zigbee network status

RED - Network link status

Cyan solid ：The device is starting up

Blue solid : USB to ZigBee mode

Blue blinking : Zigbee firmware is being updated

Green solid : ZigBeeMQTT has connected successfully

Green blinking slowly ：ZigBeeMQTT is not connected

Red solid ：Ethernet mode network disconnected

Off ： Be turned off / Device is not powered on
