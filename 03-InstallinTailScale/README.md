# Tailscale VPN + Subnet Router on Proxmox

## Project Goal

Set up **Tailscale inside a Proxmox LXC container** to securely access the homelab remotely.

The container was also configured as a **subnet router** so devices connected through Tailscale could reach the home network on:

`192.168.1.0/24`

## Environment

- Proxmox VE
- Debian LXC container
- Container ID: `102`
- Home network: `192.168.1.0/24`
- Gateway: `192.168.1.1`
- Tailscale
- Gaming PC used to manage Proxmox

## Network Layout

```text
Remote Phone / Laptop
        │
        │ Tailscale VPN
        ▼
CT 102 - Tailscale
        │
        ▼
192.168.1.0/24 Home Network
        │
        ├── Proxmox
        ├── AdGuard
        ├── Jellyfin
        └── Other Homelab Devices
```

---

## Step 1 - Create the Debian LXC Container

Created a new Debian LXC container in Proxmox for Tailscale.

### Network Configuration

```text
Bridge:        vmbr0
IPv4/CIDR:     192.168.1.60/24
Gateway:       192.168.1.1
DNS Server:    192.168.1.1
VLAN Tag:      None
Firewall:      Disabled
```

Before using `192.168.1.60`, tested the address from the main PC:

```bash
ping 192.168.1.60
```

The request timed out, showing that the address was likely available.

---

## Step 2 - Update Debian

Opened the LXC console and updated the system:

```bash
apt update
apt upgrade -y
apt install curl -y
```

---

## Step 3 - Install Tailscale

Installed Tailscale using the official install script:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

The installation completed successfully.

---

## Step 4 - Start Tailscale

Tried starting Tailscale with:

```bash
tailscale up
```

Received this error:

```text
failed to connect to local tailscaled
```

Checked the service:

```bash
systemctl status tailscaled
```

The service showed:

```text
Failed to start tailscaled.service
```

This showed that the Tailscale daemon was installed but could not start correctly inside the LXC container.

---

## Step 5 - Enable TUN Support

Tailscale requires access to the Linux TUN network device.

Stopped CT 102 from the Proxmox host:

```bash
pct stop 102
```

Opened the container configuration:

```bash
nano /etc/pve/lxc/102.conf
```

Added:

```text
features: nesting=1,keyctl=1
lxc.cgroup2.devices.allow: c 10:200 rwm
lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file
```

This gives the LXC container access to `/dev/net/tun`, which Tailscale needs to create its virtual network interface.

---

## Step 6 - Fix the LXC Configuration

When trying to start CT 102, Proxmox returned:

```text
unable to parse value of 'features'
value without key, but schema does not define a default key
```

The `features:` line was formatted incorrectly.

Changed it to:

```text
features: nesting=1,keyctl=1
```

Made sure only one `features:` line existed in the configuration.

Started the container again:

```bash
pct start 102
```

---

## Step 7 - Start the Tailscale Service

Inside CT 102:

```bash
systemctl restart tailscaled
```

Checked the service:

```bash
systemctl status tailscaled
```

The service was now able to run.

Started Tailscale:

```bash
tailscale up
```

Tailscale provided an authentication link.

Opened the link from the gaming PC and authorized the container.

---

## Step 8 - Verify Tailscale

Checked the connection:

```bash
tailscale status
```

Checked the Tailscale IPv4 address:

```bash
tailscale ip -4
```

The container received an address in the:

```text
100.x.x.x
```

range.

Normal ping tests may time out depending on firewall or ICMP settings, so Tailscale's built-in ping command can also be used:

```bash
tailscale ping <tailscale-ip>
```

---

## Step 9 - Enable IP Forwarding

To make CT 102 work as a subnet router, IPv4 forwarding had to be enabled.

Ran:

```bash
echo 'net.ipv4.ip_forward = 1' >> /etc/sysctl.conf
sysctl -p
```

This allows the Linux container to forward network traffic between Tailscale and the home network.

---

## Step 10 - Advertise the Home Network

The home network uses:

```text
192.168.1.0/24
```

Advertised that subnet through Tailscale:

```bash
tailscale up --advertise-routes=192.168.1.0/24
```

Opened the Tailscale admin console and approved the advertised route.

CT 102 now works as a **Tailscale subnet router**.

---

## What the Subnet Router Does

Without subnet routing, Tailscale can only directly reach devices running Tailscale.

With the subnet router enabled, remote devices can also reach other devices on the home network.

Example:

```text
Remote Laptop
      │
      ▼
Tailscale
      │
      ▼
CT 102
      │
      ├── 192.168.1.x Proxmox
      ├── 192.168.1.x AdGuard
      ├── 192.168.1.x Jellyfin
      └── Other LAN Devices
```

This allows secure remote access without exposing Proxmox or other homelab services directly to the public internet.

---

## Optional - Configure an Exit Node

The same container can also be configured as a Tailscale exit node.

Command:

```bash
tailscale up --advertise-routes=192.168.1.0/24 --advertise-exit-node
```

### Subnet Router

Used to access devices inside the home network.

### Exit Node

Routes normal internet traffic through the home internet connection.

Example:

```text
Laptop Away From Home
        │
        ▼
Tailscale
        │
        ▼
Home Exit Node
        │
        ▼
Internet
```

Websites would see the home's public IP while the exit node is being used.

---

## Problems Encountered

### Tailscale Service Would Not Start

Error:

```text
failed to connect to local tailscaled
```

Cause:

The LXC container did not have access to:

```text
/dev/net/tun
```

Fix:

Added TUN access to the Proxmox LXC configuration.

### Proxmox Features Configuration Error

Error:

```text
unable to parse value of 'features'
```

Cause:

Incorrect formatting of the `features:` line.

Fix:

Changed the configuration to:

```text
features: nesting=1,keyctl=1
```

---

## Commands Used

```bash
apt update
apt upgrade -y
apt install curl -y

curl -fsSL https://tailscale.com/install.sh | sh

systemctl restart tailscaled
systemctl status tailscaled

tailscale up
tailscale status
tailscale ip -4
tailscale ping <tailscale-ip>

echo 'net.ipv4.ip_forward = 1' >> /etc/sysctl.conf
sysctl -p

tailscale up --advertise-routes=192.168.1.0/24
```

### Proxmox Host Commands

```bash
pct stop 102
nano /etc/pve/lxc/102.conf
pct start 102
```

---

## Skills Practiced

- Proxmox VE
- LXC containers
- Debian Linux
- Static IPv4 addressing
- VPN configuration
- Tailscale
- Linux systemd services
- TUN interfaces
- IP forwarding
- Subnet routing
- Remote administration
- Network troubleshooting
- Proxmox configuration troubleshooting

---

## What Was Learned

This lab helped build a better understanding of how VPNs and routing work inside a homelab.

The biggest troubleshooting issue was getting Tailscale to run properly inside a Proxmox LXC container. The container needed access to `/dev/net/tun` before the Tailscale service could start.

The lab also showed the difference between a **Tailscale device**, **subnet router**, and **exit node**.

Instead of opening Proxmox and other homelab services directly to the internet, Tailscale provides a private encrypted connection back into the home network.

---

## Result

Successfully deployed Tailscale inside a Debian LXC container on Proxmox and configured the container to advertise the `192.168.1.0/24` home network.

This provides a foundation for securely accessing homelab services remotely without directly exposing them to the public internet.
