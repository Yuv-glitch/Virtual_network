# Linux Router/Firewall Lab

## 1. Project Overview

This project documents the process of building a functional network router and stateful firewall using a standard Ubuntu Server VM. The goal was to create a secure, private Local Area Network (LAN) with controlled internet access, simulating a common small office or home lab network setup. The firewall enforces security policies and provides internet connectivity to client machines on the private LAN through Network Address Translation (NAT).

This lab demonstrates core networking principles, including IP routing, packet filtering, and network troubleshooting at both Layer 2 and Layer 3.

---

## 2. Network Topology

The lab consists of two virtual machines operating on different network segments. The firewall VM acts as an intermediary between the public internet and the private client network.



---

## 3. Key Features & Technologies

* **Host OS:** Ubuntu Server 22.04
* **Client OS:** Kali Linux Desktop
* **Virtualization:** VirtualBox / VMware
* **Firewall:** UFW (Uncomplicated Firewall), iptables
* **Networking:** Netplan, IP Forwarding, NAT (Masquerade)
* **Core Features Implemented:**
    * **Stateful Packet Inspection (SPI) Firewall:** To monitor and control traffic based on connection states.
    * **Network Routing:** To forward packets between the public WAN and private LAN interfaces.
    * **Network Address Translation (NAT):** To allow multiple clients on the private network to share a single public IP address.
    * **Secure Network Segmentation:** To protect client machines from direct exposure to the internet.

---

## 4. Configuration Highlights

### IP Forwarding

Enabled packet forwarding in the kernel to allow the server to function as a router. This was made persistent by uncommenting the following line in `/etc/sysctl.conf`:
```ini
net.ipv4.ip_forward=1
```
### Netplan Configuration (Firewall VM)

``` ini
# /etc/netplan/00-installer-config.yaml
network:
  version: 2
  ethernets:
    # WAN Interface - connected to the internet via NAT/Bridged mode
    ens33:
      dhcp4: true

    # LAN Interface - for the private internal network
    ens34:
      dhcp4: no
      addresses:
        - 192.168.200.1/24
```
### Firewall and NAT setup (UFW)

``` ini
# Set default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw default allow routed

# Allow essential traffic
sudo ufw allow ssh

#/etc/ufw/before.rules
# NAT table rules
*nat
:POSTROUTING ACCEPT [0:0]

# Forward traffic from our LAN (192.168.200.0/24) to our WAN interface (ens33)
-A POSTROUTING -s 192.168.200.0/24 -o ens33 -j MASQUERADE

COMMIT
```

## Testing
  Create a client VM (e.g., Kali Linux).

  Connect its single network adapter to the same "Internal Network" as the firewall's LAN interface.

  Manually configure its IP settings:
  
    IP Address: 192.168.200.50
    
    Netmask: 255.255.255.0
    
    Gateway: 192.168.200.1
    
    DNS: 8.8.8.8

    
  From the client's terminal, confirm that ping 8.8.8.8 is successful.

