Inswift Zigbee 3.0 PoE Ethernet USB Adapter's Firmware

Zigbee over LAN,USB,WiFI is supported.

<img width="1000" height="1000" alt="07" src="https://github.com/user-attachments/assets/7fa0eef4-7823-4251-b3df-658bf69f563e" />
<img width="1000" height="1000" alt="08" src="https://github.com/user-attachments/assets/53e889e3-4b1c-41e2-9617-d5ebcaed8c95" />

1.SOC INFO 

used ESP32 + TI CC2652P2/P5 or Silicon Labs MG21/24

Performance ranking: EFR32MG24 > EFR32MG21 > CC652P7 > CC2652P

Compatibility ranking: CC2562P > EFR32MG24 = EFR32MG21 > CC2652P

High performance: EFR32 supports multiple protocols such as Matter and Thread

CC2652P： 
8MHz Cortex-M4 + 88KB RAM, the Zigbee firmware theoretically supports 200 endpoints 
and anti-interference capability,making it suitable for medium to large-scale networks. 
It requires configuration of "adapter:zstack", offers high cost-performance, and is fully supported by Zigbee3.0/Zigbee3.2.

CC2652R： 
It has the same processor as the CC2652P (48MHz Cortex-M4), but the RAM is increased to 152KB,
and it can therefore reduce the running delay. The firmware does not support Thread, and the device
has low power consumption during sleep. It can run Thread and Zigbee end devices, and supports advanced
functions such as Mobile Target Detection (MTD) (ITZH). It requires more RAM, and its performance in complex
scenarios will be limited. It fully supports Zigbee3.0/Zigbee3.2.

EFR32MG21： 
80MHz ARM Cortex-M33 processor + 64KB RAM, with computing power superior to TI chips, 
supports AES-128 hardware encryption, 802.15.4e TSCH mode, 2.4GHz transmission power of +10dBm,
-104dBm receive sensitivity, and supports Zigbee Quality of Service (QoS) and Mesh networking functions.

EFR32MG24：
78MHz Cortex-M33 processor + 552KB RAM, boasting the strongest performance, integrating TrustZone security, 
19.5dBm transmit power and 9.5dBm receive sensitivity, delivering optimal signal coverage and anti-interference capability, 
supporting hardware encryption Secure Vault and multiple protocols (Matter/Thread/Zigbee), and suitable for enterprise-level or complex IoT applications.
<img width="773" height="535" alt="image" src="https://github.com/user-attachments/assets/7df2ee49-3107-4df1-b24a-7d636fb26d1d" />



2.LED status

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


