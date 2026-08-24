# Proxmox VE Homelab Server

**Date:** August 20, 2026  
**Status:** ✅ Completed

## Project Goal

For this project, I installed **Proxmox VE** on a **MINISFORUM UM870 Slim** to create a dedicated virtualization server for my homelab.

The project involved preparing installation media, working with BIOS/UEFI settings, installing the hypervisor, configuring network access, and troubleshooting problems with the installation USB.

---

## Lab Hardware

| Device | Role |
|---|---|
| MINISFORUM UM870 Slim | Proxmox virtualization server |
| USB Flash Drive | Proxmox installation media |
| UniFi Flex 2.5G Switch | Network connectivity |
| Main Windows PC | Remote management of Proxmox |

---

## Installation Process

I started by downloading the **Proxmox VE ISO installer** from the official Proxmox website and flashing it to a USB drive to create the installation media.

Next, I entered the MINISFORUM BIOS/UEFI by pressing the **Delete** key during startup. From there, I selected the USB flash drive as the boot device and disabled Secure Boot.

Once the correct installation media was working, I booted into the Proxmox installer and installed Proxmox VE onto the system's internal **NVMe SSD**.

After installation, I configured the Proxmox management interface with an IP address so I could access and manage the server from another computer on my home network.

---

# Troubleshooting

## Problem 1 — System Booted Into Windows

My first attempt did not boot into the Proxmox installer. Instead, the computer continued booting into Windows.

I worked through several possible causes:

- Verified that the USB flash drive appeared as a boot device.
- Selected the USB flash drive as the primary boot device.
- Disabled Secure Boot.
- Recreated the installation USB.
- Attempted to boot from the USB again.

During this process, I discovered that I had accidentally downloaded an **ARM image instead of the appropriate x86-64 Proxmox VE ISO installer** for the MINISFORUM system.

The ARM image was the wrong CPU architecture for this computer, so it was not the correct installer for the UM870 Slim.

I downloaded the proper ISO, recreated the installation media, and was then able to boot into the Proxmox installer.

---

## Problem 2 — Preparing the USB Again

After the unsuccessful installation attempts, I used **DiskPart in Windows** to clean and reformat the USB flash drive before recreating the Proxmox installation media.

I opened Command Prompt and launched DiskPart:

```text
diskpart
```

I displayed the connected disks:

```text
list disk
```

The USB flash drive was listed as **Disk 2**, so I selected it:

```text
select disk 2
```

Before making changes, I verified the selected disk:

```text
detail disk
```

I then removed the existing partition information:

```text
clean
```

Created a new primary partition:

```text
create partition primary
```

Formatted it as exFAT:

```text
format fs=exfat quick
```

Assigned drive letter `D`:

```text
assign letter=D
```

Then exited DiskPart:

```text
exit
```

> **Important:** The `clean` command erases the partition information on the selected disk. I verified that I had selected the USB flash drive before running this command.

With the USB prepared again, I recreated the installation media using the correct Proxmox ISO.

---

# Final Result

The installation was successful, and the **MINISFORUM UM870 Slim is now running Proxmox VE as my homelab virtualization server**.

I configured its management network with an IP address and connected the server to my home network through the UniFi switch.

I can now administer Proxmox remotely from my main Windows PC.

---

## Verification

I verified the completed setup by:

- Confirming that Proxmox VE booted successfully.
- Confirming that the server had network connectivity.
- Connecting the server to my network through the UniFi switch.
- Opening the **Proxmox VE web management interface** from my main PC.

Being able to reach the Proxmox web interface remotely confirmed that both the installation and management network were functioning.

---

## Skills and Lessons

This project gave me hands-on practice with:

- Type-1 hypervisor installation
- Bootable USB installation media
- BIOS/UEFI configuration
- Boot-device selection
- Secure Boot basics
- Windows DiskPart
- Disk partitioning and formatting
- Basic IPv4 network configuration
- Remote server management
- Troubleshooting installation media
- CPU architecture compatibility

I already had some experience working with bootable USB drives and BIOS/UEFI settings from building PCs. This project gave me an opportunity to apply that experience to deploying and configuring a dedicated virtualization server.

---

## Next Steps

Now that the Proxmox host is operational, I plan to continue using it for projects involving:

- Virtual machines
- Linux containers
- Networking
- DNS services
- Server administration
- Storage
- IT troubleshooting
