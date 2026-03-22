# Dual-Boot-Recovery
# 🔧 Dual-Boot Failure Recovery & EFI Reconfiguration (Windows 11 + Pop!_OS)

## 📌 Executive Summary

This project documents the recovery of a failed Windows bootloader in a dual-boot Linux environment, caused by conflicting EFI partitions and corrupted/missing boot files.

The system was restored **without OS reinstallation or data loss** by rebuilding the Windows Boot Manager from a Linux-prepared recovery environment.

Additionally, the root cause (multi-EFI fragmentation) was identified, with a plan for long-term remediation.

---

## 🧠 Problem Statement

A Lenovo-based UEFI system running a dual-boot configuration encountered the following issues:

* System booted exclusively into Pop!_OS
* Windows Boot Manager was not accessible from firmware
* EFI partition (`p1`) lacked valid Windows boot files
* Multiple EFI partitions (`p1`, `p6`) caused inconsistent boot behavior

---

## 🖥️ System Architecture

### Disk Layout (NVMe)

```
nvme0n1
├── p1  (260MB)  → Windows EFI (corrupted)
├── p2  (16MB)   → Reserved
├── p3  (251GB)  → Windows OS (C:)
├── p4  (200GB)  → Data (D:)
├── p6  (1GB)    → Pop!_OS EFI (active)
├── p7/p8/p9     → Linux + recovery
```

---

## ⚠️ Root Cause Analysis

### Primary Issue

* Windows EFI partition (`p1`) was missing critical boot files:

  ```
  \EFI\Microsoft\Boot\bootmgfw.efi
  ```

### Contributing Factors

* Dual EFI partitions:

  * Windows → `p1`
  * Pop!_OS → `p6`
* Pop!_OS uses `systemd-boot` (does not auto-detect external OS)
* Firmware prioritized Pop!_OS boot entry

### Result

* Windows installation remained intact
* Boot chain was broken at EFI level

---

## 🔍 Diagnostic Workflow

### 1. Inspect Block Devices

```bash
lsblk
```

### 2. Inspect UEFI Entries

```bash
sudo efibootmgr
```

### Key Observations

* Windows Boot Manager entry existed but failed
* Boot order prioritized Pop!_OS
* EFI partition appeared empty when mounted

---

## 🛠️ Recovery Strategy

Rebuild Windows bootloader using Windows Recovery Environment (WinRE), without reinstalling the OS.

---

## 💽 Step 1: Create Bootable Windows USB (from Linux)

Using Ventoy:

```bash
wget https://github.com/ventoy/Ventoy/releases/download/v1.1.05/ventoy-1.1.05-linux.tar.gz
tar -xvf ventoy-1.1.05-linux.tar.gz
cd ventoy-1.1.05
sudo ./Ventoy2Disk.sh -i /dev/sdX
```

Copy ISO:

```bash
cp Win11.iso /mnt/ventoy/
```

---

## 🚀 Step 2: Boot into Windows Recovery

* Boot via BIOS (F12)
* Select USB
* Navigate:

  * Repair your computer
  * Troubleshoot → Advanced Options → Command Prompt

---

## 🔧 Step 3: Partition Identification

```cmd
diskpart
list disk
select disk 0
list partition
```

### Target Partitions

* EFI → Partition 1 (~260MB)
* Windows → Partition 3 (~251GB)

---

## 🔑 Step 4: Assign Temporary Mount Points

```cmd
select partition 1
assign letter=S

select partition 3
assign letter=C
exit
```

---

## ✅ Step 5: Validate Windows Filesystem

```cmd
dir C:\Windows
```

---

## 🔥 Step 6: Rebuild Windows Bootloader

```cmd
bcdboot C:\Windows /s S: /f UEFI
```

### Result

```
Boot files successfully created.
```

---

## 🔁 Step 7: System Recovery

* Removed installation media
* Rebooted system
* Windows boot restored successfully

---

## 🔄 Boot Flow Comparison

### ❌ Before Fix

```
UEFI → Pop_OS EFI (p6) → systemd-boot → Pop_OS only
```

### ✅ After Fix

```
UEFI → Windows EFI (p1) → Windows Boot Manager → Windows
```

---

## 🧠 Key Technical Insights

* Windows OS integrity is independent of EFI bootloader state
* EFI partition corruption is a common failure point in dual-boot systems
* `bcdboot` is the most reliable method for rebuilding Windows boot configuration
* Multiple EFI partitions introduce ambiguity at firmware level
* systemd-boot does not scan external EFI partitions

---

## ⚙️ Post-Recovery Remediation Plan

### Goal: Single EFI Architecture

```
Target State:
EFI (p1)
├── Microsoft/
├── systemd/
├── loader/
```

### Benefits

* Deterministic boot behavior
* Simplified firmware configuration
* Reduced risk of future boot failure

---

## 📈 Skills Demonstrated

* UEFI Boot Process Understanding
* Low-Level System Recovery
* Cross-Platform Debugging (Linux ↔ Windows)
* Disk & Partition Analysis
* Bootloader Reconstruction
* Production-Style Incident Resolution

---

## 🏁 Outcome

* Windows boot functionality fully restored
* Zero data loss
* Root cause identified
* System stability improved

---
