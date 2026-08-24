# AdGuard Home DNS Server

**Date:** August 22, 2026  
**Status:** Completed.

## Project Goal

Install and configure **AdGuard Home** in the Proxmox homelab to create a DNS server that can block ads and trackers on personal devices.

This project provided hands-on experience with DNS, Linux containers, static IPv4 addressing, Linux administration, and network troubleshooting.

---

## Hardware Used

- MINISFORUM UM870 Slim
- UniFi Flex 2.5G Switch
- Main Windows PC
- Ethernet connection

## Software / Technologies Used

- Proxmox VE
- Linux LXC containers
- Debian 12
- AdGuard Home
- Windows networking tools

---

## Network Topology

The network for this project is structured approximately as:

```text id="j91d2k"
Internet
   │
   ▼
Gateway
   │
   ▼
UniFi Flex 2.5G Switch
   │
   ├────────────── Main Windows PC
   │
   └────────────── Proxmox Server
                         │
                         ▼
                  Debian 12 LXC
                         │
                         ▼
                   AdGuard Home
```

AdGuard Home runs inside its own Debian 12 LXC container, keeping the DNS service separated from the Proxmox host.

---

# Installation

## 1. Created a Debian LXC Container

Downloaded a **Debian 12 LXC template** through Proxmox.

Then selected **Create CT** and created a dedicated Linux container for the AdGuard Home server.

---

## 2. Allocated Resources

Configured the AdGuard container with:

| Resource | Allocation |
| -------- | ---------: |
| Storage  |       8 GB |
| CPU      |     1 Core |
| Memory   |     512 MB |

These resources were sufficient for the current AdGuard Home deployment.

---

## 3. Configured Networking

Assigned the AdGuard container a **static IPv4 address**.

Using a static address is important because client devices need a consistent IP address to use AdGuard Home as their DNS server.

The basic DNS path is:

```text id="dx4xg9"
Client Device
     │
     │ DNS Request
     ▼
AdGuard Home
     │
     ▼
Upstream DNS Server
```

AdGuard can then evaluate DNS requests against its configured filtering rules.

---

## 4. Tested Network Connectivity

Before installing AdGuard Home, started the container and verified that it could communicate with the home network.

Tested connectivity to the default gateway:

```bash id="pt7k69"
ping -c 4 <gateway-ip>
```

Then tested internet connectivity and DNS resolution:

```bash id="2itjdb"
ping -c 4 google.com
```

Both tests were successful.

This confirmed that the container had working network connectivity before continuing with the AdGuard installation.

---

## 5. Updated Debian

Inside the container, updated the Debian package lists:

```bash id="2sqqqg"
apt update
```

Also made sure that `curl` was installed so that the AdGuard Home installation script could be retrieved.

---

## 6. Installed AdGuard Home

Retrieved and ran the AdGuard Home installation script using `curl`.

After installation, the AdGuard Home service was running inside the Debian LXC container.

---

## 7. Configured AdGuard Home

Opened the AdGuard Home web interface from the main PC using the container's IP address.

```text id="y52pmq"
http://<adguard-ip>
```

Completed the initial setup and configured:

- Administration account
- Web administration interface
- DNS server
- DNS filtering
- Blocklists

Once configuration was complete, the AdGuard Home DNS server was operational.

---

## 8. Configured Windows PC

To make the Windows PC use AdGuard Home, manually changed the DNS configuration on the Windows Ethernet adapter.

Set:

```text id="vv4lyc"
Preferred DNS Server: <AdGuard-IP>
```

This directed DNS requests from the computer to the AdGuard Home server.

---

# Issues and Troubleshooting

## Windows Was Using IPv6 DNS Instead of AdGuard

After setting the Windows preferred DNS server to the AdGuard Home IPv4 address, noticed that DNS requests were not always going through AdGuard.

Tested DNS resolution with:

```cmd id="e1r6a8"
nslookup google.com
```

The results showed that Windows was using an **IPv6 DNS server** instead of the IPv4 address assigned to the AdGuard Home server.

### Investigating the Problem

Checked the computer's complete network configuration using:

```cmd id="wh7vpd"
ipconfig /all
```

This showed that the computer had both the AdGuard IPv4 DNS address and an IPv6 DNS server available.

Then queried AdGuard Home directly:

```cmd id="q8tb5v"
nslookup google.com <AdGuard-IP>
```

The query was successful.

This was an important troubleshooting step because it confirmed that **AdGuard Home itself was functioning correctly**. The problem was with which DNS server the Windows client was choosing.

### Testing a Solution

As a troubleshooting test, disabled IPv6 on the Windows Ethernet adapter so Windows would use the configured IPv4 DNS server.

Then cleared the Windows DNS resolver cache:

```cmd id="73dm13"
ipconfig /flushdns
```

Finally, tested DNS resolution again:

```cmd id="2r4wwm"
nslookup google.com
```

This time the results showed that the DNS request was being handled by the AdGuard Home server.

---

# Verification

Verified the completed deployment by:

- Confirming the Debian LXC container was running.
- Confirming the container could reach the network gateway.
- Confirming the container had internet and DNS connectivity.
- Accessing the AdGuard Home web interface.
- Querying the AdGuard DNS server directly with `nslookup`.
- Verifying that Windows was using AdGuard for DNS.
- Monitoring DNS requests from the AdGuard Home dashboard.
- Confirming that filtering rules were blocking DNS requests.
- Configuring two PCs to use the AdGuard Home server.

---

# Final Result

AdGuard Home was successfully deployed inside a **Debian 12 LXC container running on Proxmox VE**.

Currently have **two Windows PCs using the AdGuard Home server for DNS filtering**.

DNS queries and filtering activity can be monitored through the AdGuard Home web interface while keeping the DNS service isolated from the Proxmox host in its own LXC container.

---

# Skills Practiced

This project provided hands-on experience with:

- Proxmox VE
- Linux LXC containers
- Debian administration
- Installing and managing Linux services
- Static IPv4 addressing
- Basic subnet/CIDR concepts
- Default gateways
- DNS servers
- DNS filtering
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

---

# What Was Learned

One of the biggest things learned from this project was how to **troubleshoot a network service instead of assuming the service itself was broken**.

When AdGuard did not initially appear to be handling the computer's DNS requests, worked through the problem by checking the Windows network configuration, testing DNS resolution, querying the AdGuard server directly, and comparing the results.

The direct `nslookup` test showed that AdGuard Home was functioning correctly and helped narrow the issue down to the client-side DNS configuration.

This project helped provide a better understanding of how **DNS clients, DNS servers, IPv4/IPv6, and network configuration work together**.

---

## Future Improvements

Future improvements for the homelab include:

- Configure additional devices to use AdGuard Home.
- Learn more about IPv6 DNS configuration.
- Explore router-level DNS configuration for network-wide filtering.
- Learn more about DHCP and DNS integration.
- Experiment with additional filtering and monitoring.
- Continue documenting troubleshooting and configuration changes.
