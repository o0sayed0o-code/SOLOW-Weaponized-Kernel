# SOLOW Weaponized Kernel (Z Fold 3)
### Custom Snapdragon 888 Kernel with Native Monitor Mode & Packet Injection (RTL8192EU & Atheros AR9271) for Samsung Galaxy Z Fold 3 (SM-F926B)

---

## Overview
This repository hosts the custom-compiled Linux kernel (GKI 5.4.274-qgki) and out-of-tree drivers optimized for security auditing, wireless penetration testing, and kernel-level subsystem research on the Samsung Galaxy Z Fold 3 (SM-F926B / Snapdragon 888). 

The kernel integrates native support for the Qualcomm Atheros AR9271 chipset and includes a pre-compiled, patched driver for the Realtek RTL8192EU chipset. This configuration enables native Monitor Mode and Frame Injection in Android user-space via Termux. Hardware and software constraints imposed by Samsung Knox, DEFEX LSM, and TrustZone RKP/KDP are bypassed.

---

## Key Features

*   **Pre-Rooted Boot Image:** The released `boot.img` is pre-patched with the Magisk ramdisk payload, providing root access immediately upon flashing.
*   **Built-in Atheros AR9271 Support (`CONFIG_ATH9K_HTC=y`):** The driver is compiled directly into the kernel binary (`Image`), eliminating the need to compile or manually load dependent modules (`ath`, `ath9k_hw`, `ath9k_common`, `ath9k_htc`).
*   **Out-of-Tree Realtek RTL8192EU Support:** Patched module resolving virtual filesystem (VFS) namespace restrictions and SMMU constraints under the Android GKI architecture.
*   **Knox Hardware Protection:** Patched `oemflag.c` to prevent hardware tamper fuses from burning during customized boot, preserving Knox status.
*   **DEFEX Security Bypass:** Disabled signal hooks inside `defex_main.c` to prevent process termination under security policy violations.
*   **Hypervisor/TrustZone Bypass:** Stubbed out EL3 secure monitor calls in assembly (`arch/arm64/kernel/fastuh_entry.S`) to bypass memory protection locks.

---

## Repository Structure
* `/patches/`: Kernel source code patches for custom bypasses and scheduler modifications.
* `/assets/` (Refer to Releases):
  * `boot.img`: Pre-rooted custom kernel image.
  * `AP_PATCHED.tar`: Odin-flashable archive containing the patched boot image.
  * `8192eu.ko`: Compiled Realtek driver module.
  * `htc_9271-1.4.0.fw`: Firmware binary for the Atheros AR9271 chipset.

---

## Flashing & Installation Guide

### Step 1: Kernel Installation
1. Reboot the device into **Download Mode**.
2. Launch **Odin3** on your host PC.
3. Load `AP_PATCHED.tar` into the **AP** slot.
4. Flash stock firmware files (BL, CP, CSC) alongside the AP file to maintain partition alignment.
5. Click **Start**. The device will reboot with root access enabled via the Magisk stub.

---

### Step 2: User-space Environment Setup
Open **Termux** and install the required root wireless packages:
```bash
pkg install root-repo -y
pkg update -y
pkg install iw wireless-tools tcpdump -y
```

---

### Step 3: Firmware Deployment (Atheros AR9271)
Since Android 11+ mounts `/vendor` as a read-only filesystem (EROFS), direct file placement is restricted. Use Magisk's systemless loop mount structure:

1. Copy `htc_9271-1.4.0.fw` and its fallback `htc_9271.fw` to the device's `/sdcard/Download/` directory.
2. In **Termux**, execute the following commands as root:
```bash
su

# Create the Magisk systemless directory structure
mkdir -p /data/adb/modules/solow_core/system/vendor/firmware/ath9k_htc/

# Generate the module descriptor
echo -e "id=solow_core\nname=SOLOW OS Core Integration\nversion=v1.4\nversionCode=2\nauthor=SOLOW\ndescription=Systemless firmware injector for ath9k_htc" > /data/adb/modules/solow_core/module.prop

# Place the firmware binaries
cp /sdcard/Download/htc_9271.fw /data/adb/modules/solow_core/system/vendor/firmware/
cp /sdcard/Download/htc_9271-1.4.0.fw /data/adb/modules/solow_core/system/vendor/firmware/ath9k_htc/

# Set file permissions
chmod 644 /data/adb/modules/solow_core/system/vendor/firmware/htc_9271.fw
chmod 644 /data/adb/modules/solow_core/system/vendor/firmware/ath9k_htc/htc_9271-1.4.0.fw
```
3. **Reboot the device.** Magisk will overlay the files into `/vendor/firmware/` during the early boot phase.

---

### Step 4: Loading and Interacting with Drivers

#### Qualcomm Atheros AR9271:
1. Connect the AR9271 USB adapter via a USB-C OTG cable.
2. Elevate to root in **Termux** and verify the interface:
```bash
su
export PATH=/data/data/com.termux/files/usr/bin:$PATH
iwconfig
# The interface should be visible (typically wlan1 or wlan2)
```
3. Initialize Monitor Mode:
```bash
ifconfig wlan1 down
iw dev wlan1 set type monitor
ifconfig wlan1 up
tcpdump -i wlan1 -n
```

#### Realtek RTL8192EU:
1. Copy `8192eu.ko` to the device.
2. Load the module:
```bash
su
setenforce 0
cp /sdcard/Download/8192eu.ko /data/local/tmp/
chmod 644 /data/local/tmp/8192eu.ko
insmod /data/local/tmp/8192eu.ko
```
3. Enable Monitor Mode:
```bash
export PATH=/data/data/com.termux/files/usr/bin:$PATH
ifconfig wlan1 down
iw dev wlan1 set type monitor
ifconfig wlan1 up
```

---

## Disclaimer
> [!WARNING]
> Your warranty is now void. 
> I am not responsible for bricked devices, dead SD cards, thermonuclear war, or you getting fired because the alarm app failed. Please do some research if you have any concerns about features included in this kernel before flashing it! You are choosing to make these modifications at your own risk.

---

## License
* Licensed under the [GPL-2.0 License](LICENSE).

---

## 🤝 Contribution & License
Contributions, issues, and feature requests are welcome. Feel free to open issues or pull requests.
* Licensed under the [GPL-2.0 License](LICENSE).\n
