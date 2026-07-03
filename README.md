# SOLOW Weaponized Kernel (Z Fold 3)
### Custom Snapdragon 888 Android Kernel & RTL8192EU USB Wi-Fi Injection Driver for Samsung Galaxy Z Fold 3 (SM-F926B)

---

## 🌟 Overview
This repository provides a custom compiled kernel and out-of-tree drivers engineered to bypass Samsung's high-level hardware/software locks (**Knox**, **DEFEX LSM**, and **TrustZone Hypervisor RKP/KDP**). 

The kernel is packaged alongside the **Realtek RTL8192EU** USB wireless driver, patched to bypass Android 10+ SMMU and virtual filesystem restrictions, allowing plug-and-play **Monitor Mode** and **Packet Injection** directly inside Termux on your phone.

---

## 🛡️ Core Bypasses & Implementations
1. **TrustZone Hypervisor Bypass (RKP/KDP):** Stubbed out `fastuh_call` in assembly (`arch/arm64/kernel/fastuh_entry.S`) to bypass EL3 secure monitor interventions and prevent memory protection write-blocks.
2. **DEFEX LSM Bypass:** Modified `security/samsung/defex_lsm/core/defex_main.c` to prevent process termination under security violations by stubbing out the kernel-space `SIGKILL` signals.
3. **Knox Flag Protection:** Stubbed `oem_flags_set` in `oemflag.c` to return success without writing values to the device's physical tamper fuses, safeguarding your hardware.
4. **Driver VFS Namespace Bypass:** Implemented `MODULE_IMPORT_NS(VFS_internal_...)` inside the out-of-tree driver to allow standard filesystem read-calls restricted under modern Android GKI policies.
5. **Tracepoint Makefile Fixes:** Fixed compilation warnings and trace headers missing errors by injecting the correct source tree variables in core scheduler sub-directories.

---

## 📦 Repository Structure
* `/patches/`: Raw `.patch` files to apply to your own kernel source code tree.
* `/assets/`: Contains precompiled binary releases (refer to the Release tab).
  * `boot.img`: Precompiled kernel image containing Magisk root (based on GKI 5.4.274-qgki).
  * `8192eu.ko`: Recompiled wireless module for Snapdragon 888.

---

## 🛠️ Flashing & Installation Guide

### Step 1: Flashing the Custom Kernel
1. Place the phone into **Download Mode**.
2. Flash the patched AP archive (`AP_PATCHED.tar`) using **Odin3** in the **AP slot**.
3. For GKI signature verification issues, make sure to flash a blank/disabled verification `vbmeta` in the **USERDATA** slot if required (Magisk-patched tar files have this already packaged).
4. Reboot the phone.

### Step 2: Running the Wi-Fi Adapter in Termux
1. Move the `8192eu.ko` driver to your phone's internal storage (`/sdcard/Download/`).
2. Open **Termux** and install the root wireless tools:
   ```bash
   pkg install root-repo -y
   pkg update -y
   pkg install iw wireless-tools -y
   ```
3. Run the following commands as **root** inside Termux:
   ```bash
   su
   # Disable SELinux temporarily
   setenforce 0
   # Copy to a native partition to bypass FUSE mount restrictions
   cp /sdcard/Download/8192eu.ko /data/local/tmp/
   chmod 644 /data/local/tmp/8192eu.ko
   # Load driver
   insmod /data/local/tmp/8192eu.ko
   ```

### Step 3: Triggering Monitor Mode
1. Attach your RTL8192EU USB Wi-Fi card using a USB-C OTG cable.
2. Confirm the card is detected by identifying the newly created interface:
   ```bash
   ip link show
   # Typically appears as wlan1
   ```
3. Put the interface into monitor mode and bring it up:
   ```bash
   ifconfig wlan1 down
   iw dev wlan1 set type monitor
   ifconfig wlan1 up
   ```
4. Confirm successful execution:
   ```bash
   iwconfig wlan1
   # Look for "Mode:Monitor"
   ```
5. Test using standard utilities:
   ```bash
   airodump-ng wlan1
   ```

---

## ⚠️ Disclaimer
> [!WARNING]
> Your warranty is now void. 
> I am not responsible for bricked devices, dead SD cards, thermonuclear war, or you getting fired because the alarm app failed. Please do some research if you have any concerns about features included in this kernel before flashing it! You are choosing to make these modifications at your own risk.

---

## 🤝 Contribution & License
Contributions, issues, and feature requests are welcome. Feel free to open issues or pull requests.
* Licensed under the [GPL-2.0 License](LICENSE).\n