# Firmware Upgrade Operations for eth-52P Device
The eth-52P device includes two application firmwares: the **​​web application firmware**​​ (fs.bin) and the **​​ESP SoC application firmware**​​ (ota.bin), as well as a ​​**full firmware**​​ (full.bin) that combines both.

# ​​1. Full.bin Firmware​​
**•​​Upgrade Method​​:** Currently, the full.bin firmware can only be upgraded via the **flash_download_tool** Downloadtool on a local computer.

**•​​Note​​:** Upgrading full.bin will erase the entire flash memory, restoring all device configurations to factory defaults. This method is generally.

**•recommended:** ​​only when the device malfunctions​​ and cannot operate normally.

# 2. Web-Based Online OTA Upgrade (for ota.bin)​​
**•​​Scope​​:** This method supports upgrading only the ​​**ota.bin** firmware​​ (ESP SoC application firmware).

**•​​Process​​:** The web interface displays all released firmware versions, allowing users to select the latest or a specific version for upgrade.

**•​​Recommendation​​:** This is the preferred method for most users, as the web application firmware (fs.bin) is stable and rarely modified after release.

# ​​3. Web-Based Local Upgrade (for ota.bin and fs.bin)​​
**•​​Scope​​:** This method supports upgrading both ​​**ota.bin**​​ (ESP SoC application firmware) and **​​fs.bin**​​ (web application firmware).

**•​​Process​​:** Users can download desired firmware versions (e.g., from Releases) to their local computer and perform the upgrade via the web interface.

**•​​Advantage​​:** Local upgrades are faster than online OTA upgrades.

**•​​Recommendation​​:** This method is recommended when the web-based online OTA service is unavailable (e.g., due to network issues or server unavailability).

# ​​Summary​​:

•Use flash_download_toolfor full.bin upgrades (factory reset required, last resort).

•Use web-based online OTA for ota.bin upgrades (recommended for routine updates).

•Use web-based local upgrades for ota.bin/fs.bin when online services are inaccessible (faster alternative).


# 4.Firmware Upgrade Operation Guide
(https://github.com/Inswift-dev/inswift-eth/wiki/ETH-Firmware-Upgrade-Operation-Guide)


