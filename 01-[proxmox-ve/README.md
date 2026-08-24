# AdGuard Home DNS Server

**Date:** August 22, 2026  
**Status:** Completed

## Goal

Install and configure AdGuard Home in my Proxmox homelab to create a DNS server that can block ads and trackers on my personal devices.

---

## Hardware Used

- MINISFORUM UM870 Slim
- UniFi Flex 2.5G Switch
- Main Windows PC
- Ethernet connection

## Software / Technologies Used

- Proxmox VE
- LXC containers
- Debian 12
- AdGuard Home
- Windows networking tools

---

# Steps for Installation

## 1. Created a Debian LXC Container

- Downloaded a Debian 12 LXC template through Proxmox.
- Selected **Create CT** to create a new Linux container.
- Created the container specifically for AdGuard Home.

---

## 2. Allocated Resources

Configured the AdGuard container with:

- **Storage:** 8 GB
- **CPU:** 1 Core
- **Memory:** 512 MB

---

## 3. Configured Networking

Assigned the AdGuard container a static IPv4 address so that client devices would always know where to find the DNS server.

### Network Layout

```text
Gateway
   │
   ▼
UniFi 2.5G Switch
   │
   ├── Main PC
   │
   └── Proxmox
          │
          ▼
      LXC Container
          │
          ▼
      AdGuard Home
```

---

## 4. Tested Network Connectivity

Started the container to make sure it was running.

Before installing AdGuard Home, I tested the connectivity between the LXC container and my home network.

I tested connectivity to the gateway by using:

```bash
ping -c 4 <gateway-ip>
```

I also tested internet connectivity and DNS by using:

```bash
ping -c 4 google.com
```

Both tests were successful.

---

## 5. Updated Debian

Inside the container, I updated Debian using:

```bash
apt update
```

I also made sure `curl` was installed so I could retrieve the AdGuard Home installation script.

---

## 6. Installed AdGuard Home

I ran the `curl` installation script to install AdGuard Home inside the Debian 12 LXC container.

---

## 7. Configured AdGuard Home

I opened the AdGuard Home setup page by entering the container's IP address into my web browser:

```text
http://<AdGuard-IP>
```

I completed the installation and configured:

- Admin web interface
- DNS server
- Administration account
- DNS filtering

After completing the configuration, the AdGuard Home DNS server was up and running using the container's static IP address.

---

## 8. Configured My Main PC

I changed the DNS configuration on my Windows Ethernet adapter and set the **Preferred DNS server** to the AdGuard Home IP address.

This caused Windows to send DNS requests to my AdGuard Home server.

---

# Issues and Troubleshooting

## Windows Was Using an IPv6 DNS Server

After changing the Preferred DNS server to the AdGuard IPv4 address, Windows was still prioritizing an IPv6 DNS server instead of my AdGuard Home IPv4 address.

I ran:

```cmd
nslookup google.com
```

The results showed that the DNS request was going to an IPv6 DNS server instead of AdGuard Home.

I checked the network configuration using:

```cmd
ipconfig /all
```

This showed that my PC had both the AdGuard IPv4 DNS address and an IPv6 DNS server configured.

I then tested AdGuard Home directly by using:

```cmd
nslookup google.com <AdGuard-IP>
```

This confirmed that AdGuard Home itself was working properly.

To troubleshoot the issue, I temporarily disabled IPv6 on my Ethernet adapter so Windows would use the configured IPv4 DNS server.

I then cleared the DNS cache using:

```cmd
ipconfig /flushdns
```

After that, I ran:

```cmd
nslookup google.com
```

The result showed that my computer was now using the AdGuard Home DNS server and was no longer showing the other DNS server.

---

# Results

AdGuard Home was successfully installed in my Debian 12 LXC container on Proxmox.

I currently have **two PCs running through my AdGuard Home DNS server**.

I can monitor DNS requests through the AdGuard Home dashboard and see requests being processed and blocked.

AdGuard Home is also separated from the Proxmox host by running inside its own LXC container.

---

# Verification

I verified that AdGuard Home was working by:

- Confirming that the LXC container was running.
- Testing connectivity to my gateway.
- Testing internet connectivity.
- Opening the AdGuard Home web interface.
- Using `nslookup` to test DNS resolution.
- Using `nslookup` to directly query the AdGuard Home server.
- Checking the AdGuard Home dashboard for DNS requests.
- Confirming that two PCs could use AdGuard Home for DNS.

---

# What I Learned

Through this project, I gained experience with:

- Proxmox VE
- Linux LXC containers
- Debian administration
- Installing and managing Linux services
- Static IPv4 addressing
- Basic subnet/CIDR notation
- Default gateways
- DNS servers
- Linux package management with `apt`
- Using `curl`
- Windows network adapter configuration
- `ipconfig`
- `nslookup`
- `ping`
- DNS cache flushing
- Client/server networking
- Network troubleshooting
- Verifying services using logs

One of the main things I learned from this project was **how to troubleshoot a service**.

When AdGuard Home did not appear to be working correctly, I used different networking commands to determine whether the problem was with AdGuard itself or with my Windows DNS configuration.

By directly testing the AdGuard server with `nslookup`, I was able to confirm that the server was working and narrow the problem down to the client-side DNS configuration.
