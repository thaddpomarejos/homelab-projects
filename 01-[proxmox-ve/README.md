# Proxmox VE Homelab Server

**Date:** August 20, 2026  
**Status:** Completed

## Goal

Installing Proxmox on the **MINISFORUM UM870 Slim** and turning it into a VM server.

---

## Hardware Used

- MINISFORUM UM870 Slim
- USB flash drive
- UniFi Flex 2.5G switch
- Main PC

---

# Steps for Installation

## 1. Downloaded Proxmox VE

- Downloaded the Proxmox VE ISO from their website.
- Flashed it onto the USB drive.

## 2. Configured BIOS

- Entered BIOS by holding the **Delete** key and going through the settings.
- Disabled Secure Boot.
- Selected the USB flash drive as the boot device.

## 3. Installed Proxmox VE

- Installed Proxmox VE onto the NVMe SSD.
- Configured the IP to connect to Proxmox from another computer.

---

# Issues and Resolutions

## Issue 1 — PC Booted Into Windows Instead of Proxmox

Tried to boot into the Proxmox VE installer, but the PC booted into Windows instead of Proxmox.

### Troubleshooting the Issue

Checked the USB boot device, selected the USB flash drive to be the main boot drive, disabled Secure Boot, recreated the installation media on the USB drive, and then tried to boot into Proxmox again.

The issue encountered while trying to install Proxmox was accidentally downloading an **ARM file instead of the correct ISO file**, which made the installation process harder.

The file was the wrong type, used **ARM** file rather than the **ISO** file, wasn't the correct installer image file for the PC.

---

## Issue 2 — Reformatting the USB Drive

**DISKPART**

### Steps Used in DiskPart

Opened Command Prompt using:

```text
Win + R
```

Opened DiskPart and listed all of the disks/drives:

```cmd
diskpart
list disk
```

The USB drive was listed as **Disk 2**, so Disk 2 was selected using:

```cmd
select disk 2
```

Used the following command to verify that the correct disk was selected:

```cmd
detail disk
```

Erased the partitions using:

```cmd
clean
```

Created a new partition using:

```cmd
create partition primary
```

Formatted the drive as exFAT using:

```cmd
format fs=exfat quick
```

Assigned the USB drive a new drive letter:

```cmd
assign letter=D
```

Then exited DiskPart:

```cmd
exit
```

---

# Results

Proxmox VE was successfully installed.

Set up the management interface with an IP address and connected to it through the main PC.

---

# Verification

Successfully opened the **Proxmox VE web interface** from the main PC.

This confirmed that the Proxmox server was installed and accessible over the network.

---

# What Was Learned

Learned how to create a bootable USB installation drive. Had some prior knowledge of this from building PCs.

Also gained more experience using **BIOS/UEFI**, with some previous experience from building PCs.

Through this project, practiced:

- Creating bootable USB installation media
- Using BIOS/UEFI
- Changing boot devices
- Disabling Secure Boot
- Learning Secure Boot basics
- Using Windows DiskPart
- Reformatting a USB drive
- Installing Proxmox VE
- Configuring the Proxmox management IP
- Accessing a server through another computer
- Troubleshooting USB boot problems


