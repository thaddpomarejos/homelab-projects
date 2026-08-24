# Proxmox VE Homelab Server

**Date:** August 20, 2026  
**Status:** Completed

## Goal

Installing Proxmox on my **MINISFORUM UM870 Slim** and turning it into a VM server.

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
- Configured the IP so I could connect to Proxmox from another computer.

---

# Issues and Resolutions

## Issue 1 — PC Booted Into Windows Instead of Proxmox

I tried to boot into the Proxmox VE installer, but the PC booted into Windows instead of Proxmox.

### Troubleshooting the Issue

I checked the USB boot device, selected the USB flash drive to be the main boot drive, disabled Secure Boot, recreated the installation media on the USB drive, and then tried to boot into Proxmox again.

The issue that I encountered while trying to install Proxmox was that I accidentally downloaded an **ARM file instead of the correct ISO file**, which made the installation process harder.

It was the wrong image file for my system and wasn't the correct installer for my PC.

---

## Issue 2 — Reformatting the USB Drive

I also went into **DiskPart** using Windows Command Prompt and reformatted the USB drive because it wasn't working correctly after I flashed the wrong ARM file onto it.

### Steps I Used in DiskPart

I opened Command Prompt using:

```text
Win + R
```

I then opened DiskPart and listed all of the disks/drives:

```cmd
diskpart
list disk
```

My USB drive was listed as **Disk 2**, so I selected Disk 2 using:

```cmd
select disk 2
```

I used the following command to verify that I selected the correct disk:

```cmd
detail disk
```

I erased the partitions using:

```cmd
clean
```

I created a new partition using:

```cmd
create partition primary
```

I formatted the drive as exFAT using:

```cmd
format fs=exfat quick
```

I assigned the USB drive a new drive letter:

```cmd
assign letter=D
```

Then I exited DiskPart:

```cmd
exit
```



# Results

Proxmox VE was successfully installed.

I set up the management interface with an IP address and connected to it through my main PC.

---

# Verification

I was able to open the **Proxmox VE web interface** from my main PC.

This confirmed that the Proxmox server was installed and accessible over my network.

---

# What I Learned

I learned how to create a bootable USB installation drive. I had some prior knowledge of this from building PCs.

I also got more experience using **BIOS/UEFI**, which I had some previous experience with from building PCs.

Through this project, I practiced:

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

One of the main things I learned was how to troubleshoot installation and USB boot problems instead of starting over without knowing what caused the issue.
