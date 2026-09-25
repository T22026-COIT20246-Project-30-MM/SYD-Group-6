# Network Setup

COIT20246 Cyber Security and Networking — Term 2, 2026
SYD Group 6 — Atikur Rahman Mimmoy (12327451), Md Arman Joarder (12312653)

## 1. Assumptions

The scenario does not state every detail about the business, so we have made the following assumptions. Our network design, firewall rules and risk assessment are all based on them.

### 1.1 Location

The business is located in Sydney, New South Wales, in a leased office on Level 2 of a small commercial building in Parramatta.

We assume a single office in one location. This means the business needs only one local network, one internet connection and one router/firewall, with no links to branch offices. All staff work from this office, so we do not design for remote access or VPN connections.

### 1.2 Type of Professional Services

The business is an accounting and taxation practice. It prepares individual and small business tax returns, lodges BAS statements, provides bookkeeping and payroll services, and gives financial planning advice to local clients.

This assumption matters for security because the business stores tax file numbers, bank account details, payroll records and financial statements belonging to its clients. This data is attractive to attackers because it can be used for identity theft and fraud, and the business is legally required to protect it under Australian privacy law. For this reason confidentiality is the highest priority in our risk assessment.

### 1.3 Number of Staff and Their Roles

The business has 5 staff:

| Role | Number | Network access needed |
|---|---|---|
| Owner / principal accountant | 1 | Full access to client records and financial systems |
| Accountants | 2 | Access to client records for their own clients |
| Administrative staff | 1 | Appointments, invoicing and general correspondence |
| IT support contractor (part-time, on site weekly) | 1 | Router and server administration |

This gives 5 Windows workstations on the internal network, plus a shared network printer. Only the IT support contractor needs to reach the router management interface, which is why we restrict management access rather than allowing it from every workstation on the network.

### 1.4 Website Content

The website is a public information site only. It shows the business name, a description of the services offered, the office address and opening hours, and a contact phone number and email address.

The website does not have client logins, file uploads, online payments, or a database, and it does not collect or store any personal information. We assume clients send documents by other means and do not upload them through the site.

This assumption keeps the web server simple, since no database or user authentication is required. The website still needs protection: if it were defaced or taken offline, the business would lose credibility with existing clients and would appear untrustworthy to potential new clients, which is a reputational risk rather than a data breach risk.

## 2. Lab Network — OpenWRT and VirtualBox

This section documents the lab network we built using the OpenWRT VM provided in this unit, running in VirtualBox on a Windows host.

### 2.1 OpenWRT Version

We confirmed the identity of the VM provided in this unit with `cat /etc/openwrt_release` and `uname -a`.

![OpenWRT version](images/openwrt-version.png)

| Item | Value |
|---|---|
| Distribution | OpenWrt |
| Release | 22.03.3 |
| Revision | r20028-43d71ad93e |
| Target | x86/64 |
| Architecture | x86_64 |
| Kernel | Linux 5.10.161 #0 SMP Tue Jan 3 00:24:21 2023 x86_64 |
| Hostname | OpenWrt |

All configuration and testing in this report was carried out on this VM.

### 2.2 Interfaces and IP Addresses

We ran `ip addr` on the OpenWRT VM to list every interface and its address.

![ip addr output](images/ip-addr.png)

| Interface | IP address | MAC address | VirtualBox adapter | Purpose |
|---|---|---|---|---|
| `lo` | 127.0.0.1/8 | — | — | Loopback, used only for traffic within the VM itself. |
| `eth0` | *(none)* | 08:00:27:e4:b4:9d | Host-only Adapter | The physical port connecting to the Windows host. It has no address of its own because it is a member of the `br-mng` bridge. |
| `eth1` | 10.0.3.15/24 | 08:00:27:59:19:60 | NAT | The WAN side. Provides outbound internet access through the Windows host's connection. |
| `br-mng` | 192.168.56.2/24 | 08:00:27:e4:b4:9d | *(bridge over eth0)* | The management bridge. This is the address the Windows host uses to reach OpenWRT for the website, SSH, ping and the management interface. |

**Why `eth0` has no IP address.** `eth0` and `br-mng` share the same MAC address, `08:00:27:e4:b4:9d`. This shows that `eth0` is not an independent interface but a port bridged into `br-mng`. In a Linux bridge the member ports operate at layer 2 and carry no IP address of their own; the bridge interface holds the layer 3 address for the whole bridge. All traffic arriving on `eth0` from the Windows host is therefore handled by `br-mng` at 192.168.56.2.

**The two networks.** The VM has two separate networks, each with a different job:

- **10.0.3.0/24 on `eth1` (NAT)** — outbound internet access, used for downloading packages with `opkg`. VirtualBox assigns `10.0.x.15` to NAT adapters, which identifies this as the NAT network.
- **192.168.56.0/24 on `br-mng` (Host-only)** — the internal network between OpenWRT and the Windows host. This is the network that represents the small business LAN in our design, and every test in this report is performed over it.

### 2.3 Routing

We checked the routing table with `ip route`:

```
default via 10.0.3.2 dev eth1  src 10.0.3.15
10.0.3.0/24 dev eth1 scope link  src 10.0.3.15
192.168.56.0/24 dev br-mng scope link  src 192.168.56.2
```

![ip route output](images/ip-route.png)

This confirms how the two networks are used. The **default route** — the path for any traffic that is not destined for a directly connected network — goes out through `eth1` to 10.0.3.2, which is the VirtualBox NAT gateway. All internet-bound traffic therefore leaves over NAT. The 192.168.56.0/24 network is reached directly through `br-mng` with no gateway, because the Windows host is on the same subnet.

### 2.4 VirtualBox Adapter Configuration

The VM is named "COIT20246 OpenWRT T2 2023" in VirtualBox, which is the image provided in this unit. Its network settings are:

| Adapter | Attached to | Name | MAC address | OpenWRT interface |
|---|---|---|---|---|
| Adapter 1 | Host-only Adapter | VirtualBox Host-Only Ethernet Adapter | 08:00:27:E4:B4:9D | `eth0`, bridged into `br-mng` |
| Adapter 2 | NAT | — | 08:00:27:59:19:60 | `eth1` |

![VirtualBox Adapter 1 — Host-only](images/virtualbox-adapter1.png)

![VirtualBox Adapter 2 — NAT](images/virtualbox-adapter2.png)

**Matching the adapters to the interfaces.** We identified which VirtualBox adapter corresponds to which OpenWRT interface by comparing MAC addresses rather than assuming an order. Adapter 1 is configured with MAC `080027E4B49D`, which is the same MAC reported by `eth0` and by `br-mng` in the `ip addr` output. This confirms that Adapter 1, the host-only adapter, is the interface presented to OpenWRT as `eth0` and bridged into `br-mng` at 192.168.56.2. Adapter 2 is therefore `eth1`, which holds 10.0.3.15 — an address in the range VirtualBox uses for NAT.

### 2.5 How the Windows Host Connects to OpenWRT

The Windows host connects to OpenWRT over the **host-only network**, 192.168.56.0/24. VirtualBox creates a virtual adapter on the Windows host itself that sits on this network, so the host and the OpenWRT VM are on the same subnet and can reach each other directly. Running `ipconfig` on Windows shows this adapter, listed as "Ethernet adapter Ethernet 2", holding **192.168.56.1** with a 255.255.255.0 mask and no default gateway. OpenWRT holds **192.168.56.2** on `br-mng`.

The absence of a default gateway on that Windows adapter is expected and confirms the network is host-only: it exists purely to connect the host to the VM, and Windows continues to reach the internet through its Wi-Fi adapter on a separate network instead.

The NAT network on `eth1` works differently. NAT allows OpenWRT to make outbound connections to the internet through the Windows host's own connection, but the Windows host cannot open a connection inwards to the VM across NAT, and the 10.0.3.15 address is not reachable from the host. This is the reason all of our testing — loading the website, connecting over SSH, sending ping requests and reaching the management interface — is carried out over the host-only network at 192.168.56.2 rather than over NAT.

![Windows ipconfig](images/windows-ipconfig.png)

### 2.6 Lab Network Diagram

The diagram below shows our lab setup: the OpenWRT VM, the Windows host, each interface, the IP addresses and the VirtualBox adapter type used by each.

![Lab network diagram](images/lab-network-diagram.png)

Source file: [`images/lab-network-diagram.drawio`](images/lab-network-diagram.drawio)

### 2.7 Lab IP Address Allocation

| Device | Interface | IP address | Mask | Network | Adapter type |
|---|---|---|---|---|---|
| OpenWRT VM | `eth1` | 10.0.3.15 | 255.255.255.0 | 10.0.3.0/24 | NAT |
| OpenWRT VM | `br-mng` (bridging `eth0`) | 192.168.56.2 | 255.255.255.0 | 192.168.56.0/24 | Host-only |
| OpenWRT VM | `eth0` | *(no address — bridge member)* | — | 192.168.56.0/24 | Host-only |
| Windows host | VirtualBox Host-Only Adapter ("Ethernet 2") | 192.168.56.1 | 255.255.255.0 | 192.168.56.0/24 | Host-only |
| NAT gateway (VirtualBox) | — | 10.0.3.2 | 255.255.255.0 | 10.0.3.0/24 | NAT |
