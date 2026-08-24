# AdGuard Home DNS Server

**Date:** August 22, 2026  
**Status:** Completed.

## Project Goal

Install and configure **AdGuard Home** in my Proxmox homelab to create a DNS server that can block ads and trackers on my personal devices.

This project gave me hands-on experience with DNS, Linux containers, static IPv4 addressing, Linux administration, and network troubleshooting.

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

My network for this project is structured approximately as:

```text
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

I downloaded a **Debian 12 LXC template** through Proxmox.

I then selected **Create CT** and created a dedicated Linux container for the AdGuard Home server.

---

## 2. Allocated Resources

I configured the AdGuard container with:

| Resource | Allocation |
|---|---:|
| Storage | 8 GB |
| CPU | 1 Core |
| Memory | 512 MB |

These resources were sufficient for my current AdGuard Home deployment.

---

## 3. Configured Networking

I assigned the AdGuard container a **static IPv4 address**.

Using a static address is important because client devices need a consistent IP address to use AdGuard Home as their DNS server.

The basic DNS path is:

```text
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

Before installing AdGuard Home, I started the container and verified that it could communicate with my home network.

I tested connectivity to my default gateway:

```bash
ping -c 4 <gateway-ip>
```

I then tested internet connectivity and DNS resolution:

```bash
ping -c 4 google.com
```

Both tests were successful.

This confirmed that the container had working network connectivity before I continued with the AdGuard installation.

---

## 5. Updated Debian

Inside the container, I updated the Debian package lists:

```bash
apt update
```

I also made sure that `curl` was installed so that I could retrieve the AdGuard Home installation script.

---

## 6. Installed AdGuard Home

I retrieved and ran the AdGuard Home installation script using `curl`.

After installation, the AdGuard Home service was running inside the Debian LXC container.

---

## 7. Configured AdGuard Home

I opened the AdGuard Home web interface from my main PC using the container's IP address.

```text
http://<adguard-ip>
```

I completed the initial setup and configured:

- Administration account
- Web administration interface
- DNS server
- DNS filtering
- Blocklists

Once configuration was complete, the AdGuard Home DNS server was operational.

---

## 8. Configured My Windows PC

To make my Windows PC use AdGuard Home, I manually changed the DNS configuration on the Windows Ethernet adapter.

I set:

```text
Preferred DNS Server: <AdGuard-IP>
```

This directed DNS requests from the computer to the AdGuard Home server.

---

# Issues and Troubleshooting

## Windows Was Using IPv6 DNS Instead of AdGuard

After setting the Windows preferred DNS server to the AdGuard Home IPv4 address, I noticed that DNS requests were not always going through AdGuard.

I tested DNS resolution with:

```cmd
nslookup google.com
```

The results showed that Windows was using an **IPv6 DNS server** instead of the IPv4 address assigned to my AdGuard Home server.

### Investigating the Problem

I checked the computer's complete network configuration using:

```cmd
ipconfig /all
```

This showed that my computer had both the AdGuard IPv4 DNS address and an IPv6 DNS server available.

I then queried AdGuard Home directly:

```cmd
nslookup google.com <AdGuard-IP>
```

The query was successful.

This was an important troubleshooting step because it confirmed that **AdGuard Home itself was functioning correctly**. The problem was with which DNS server the Windows client was choosing.

### Testing a Solution

As a troubleshooting test, I disabled IPv6 on the Windows Ethernet adapter so Windows would use the configured IPv4 DNS server.

I then cleared the Windows DNS resolver cache:

```cmd
ipconfig /flushdns
```

Finally, I tested DNS resolution again:

```cmd
nslookup google.com
```

This time the results showed that the DNS request was being handled by my AdGuard Home server.

---

# Verification

I verified the completed deployment by:

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

I currently have **two Windows PCs using the AdGuard Home server for DNS filtering**.

I can monitor DNS queries and filtering activity through the AdGuard Home web interface while keeping the DNS service isolated from the Proxmox host in its own LXC container.

---

# Skills Practiced

This project gave me hands-on experience with:

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

# What I Learned

One of the biggest things I learned from this project was how to **troubleshoot a network service instead of assuming the service itself was broken**.

When AdGuard did not initially appear to be handling my computer's DNS requests, I worked through the problem by checking the Windows network configuration, testing DNS resolution, querying the AdGuard server directly, and comparing the results.

The direct `nslookup` test showed that AdGuard Home was functioning correctly and helped me narrow the issue down to the client-side DNS configuration.

This project helped me better understand how **DNS clients, DNS servers, IPv4/IPv6, and network configuration work together**.

---

## Future Improvements

As I continue developing my homelab, I plan to:

- Configure additional devices to use AdGuard Home.
- Learn more about IPv6 DNS configuration.
- Explore router-level DNS configuration for network-wide filtering.
- Learn more about DHCP and DNS integration.
- Experiment with additional filtering and monitoring.
- Continue documenting troubleshooting and configuration changes.
