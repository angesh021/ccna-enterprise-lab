# CCNA Enterprise Lab – End‑to‑end network design in Packet Tracer

This repository contains a Packet Tracer project used to practise CCNA‑level
network design.  The project demonstrates how to build a small enterprise
network that supports data, voice and wireless clients, implements IPv6
addressing, provides inter‑VLAN routing and applies security policies with a
Next Generation Firewall.  The lab topology, shown below, mixes layer‑2 and
layer‑3 switching, enterprise Wi‑Fi and firewalling to simulate the kind of
networks encountered in real world deployments.



## Contents

* **`CCNA_LAB.pkt`** – the Packet Tracer file containing the complete lab.
* **`screenshot.png`** – topology diagram used in this
  README.
* **`README.md`** – this document, describing the design, configuration and
  verification steps.

## Why this lab?

This project brings together multiple CCNA topics in a coherent network
design.  It is intended as a hands‑on exercise for anyone interested in
building and troubleshooting enterprise‑style networks.  The lab demonstrates
practical skills in areas such as VLAN deployment, inter‑VLAN routing,
firewall and NAT configuration, wireless LAN deployment and the integration
of IPv4 and IPv6.  Enterprise‑class devices are used throughout—Catalyst 3560
and 2960 switches, ISR 4321 routers, a 5506‑X ASA firewall and a 2504
wireless LAN controller—to mirror the types of equipment commonly found in
production environments.  Design decisions such as separating voice and data
VLANs or using Dynamic PAT for Internet access reflect best practices in
many real networks.

## Topology overview

The network is split into a **Trusted** internal network and an **Internet**
segment.  A Catalyst 3560 multilayer switch acts as the distribution
switch and performs inter‑VLAN routing.  Two Catalyst 2960 access switches
connect end hosts via an EtherChannel trunk.  A Cisco ASA 5506‑X firewall
protects the internal network and performs Network Address Translation (NAT)
before forwarding traffic towards a Cisco ISR 4321 edge router.  Wireless
connectivity is provided by a Cisco 2504 wireless LAN controller (WLC) and two
Lightweight Access Points (LAPs).  Voice services are implemented via
Cisco 7960 IP phones that share the same access ports as PCs using the voice
VLAN feature.

### Addressing and VLANs

The following table summarises the VLANs, their purpose and addressing in the
lab.  IPv6 is used on the data VLAN to illustrate dual‑stack operation.

| VLAN | Purpose          | Subnet(s)                         | Example hosts |
|----:|------------------|-----------------------------------|---------------|
| 10  | Data (wired/wireless) | `10.0.10.0/24` (IPv4) and `2001:10::/64` (IPv6) | PCs (10.0.10.21–22) and smartphones |
| 11  | Voice            | `10.0.11.0/24`                    | IP phones |
| 12  | Wireless APs     | `10.0.12.0/24`                    | Lightweight APs and WLC |
| 13  | Server segment   | `10.0.13.0/24`                    | Internal server (10.0.13.5) |
| —   | Core links       | `10.0.255.0/30` (switch↔router), `10.0.255.4/30` (switch↔ASA) | 3560 switch (.1/.5), ISR 4321 (.2), ASA (.6) |
| —   | Internet links   | `50.1.2.0/30` (ASA↔edge router), `8.0.0.0/30` (edge router↔ISP server) | ASA (.1), edge router (.2), ISP server (.2) |

### Device roles

| Device | Function |
|---|---|
| **Catalyst 3560‑24PS** | Multilayer PoE switch; provides inter‑VLAN routing via SVIs and delivers power to IP phones and wireless access points.  It supports advanced QoS, access control lists and high‑performance IP routing while maintaining the simplicity of LAN switching:contentReference[oaicite:0]{index=0}.  The 24 Fast Ethernet ports with SFP uplinks and PoE allow the deployment of IP telephony and wireless access without additional power supplies:contentReference[oaicite:1]{index=1}. |
| **Catalyst 2960‑24TT (two units)** | Access switches that provide edge‑port connectivity to PCs, phones and APs.  They are connected to the 3560 via an EtherChannel trunk for increased bandwidth and redundancy. |
| **Cisco ISR 4321 routers (two units)** | Router 0 connects the internal network to the ASA through a /30 link.  Router 1 acts as the edge router towards the Internet.  The ISR 4321 is a versatile router that provides high‑speed connectivity and built‑in security features; it can handle large amounts of traffic without performance loss and includes intrusion prevention, firewall and VPN capabilities:contentReference[oaicite:2]{index=2}.  It is easy to set up thanks to its user‑friendly interface:contentReference[oaicite:3]{index=3}. |
| **Cisco ASA 5506‑X firewall** | Provides stateful inspection, NAT and VPN termination.  The ASA 5500‑X series combines proven firewall technology with intrusion‑prevention capabilities; it uses a 64‑bit multicore architecture that offers higher firewall and VPN throughput and can run multiple security services simultaneously:contentReference[oaicite:4]{index=4}.  The ASA supports standard security services such as Cisco Security Intelligence Operations, application visibility and control, built‑in IPS and integration with AnyConnect VPN clients:contentReference[oaicite:5]{index=5}. |
| **Cisco 2504 WLC** | Centralised wireless controller that manages the lightweight APs.  This WLC supports up to 75 access points and 1 000 clients:contentReference[oaicite:6]{index=6}.  It offers advanced security mechanisms and comprehensive management capabilities while remaining suitable for small‑to‑medium networks:contentReference[oaicite:7]{index=7}:contentReference[oaicite:8]{index=8}.  Initial configuration involves connecting the controller to the network, accessing the web interface with the `admin` / `Cisco123` credentials and registering the APs:contentReference[oaicite:9]{index=9}. |
| **Lightweight APs (2 × LAP‑PT)** | Provide wireless coverage for mobile clients.  They obtain configuration and firmware from the WLC over the management VLAN. |
| **IP phones (7960)** | Voice endpoints that connect through the access switches.  The voice VLAN feature uses Cisco Discovery Protocol version 2 (CDPv2) to tell the phone which VLAN ID to tag voice packets:contentReference[oaicite:10]{index=10}.  The switch must be configured with a voice VLAN before it can advertise it; a simple configuration uses the `vlan <id>` commands and marks the VLAN as `voice`:contentReference[oaicite:11]{index=11}. |
| **End hosts** | PCs and smartphones on the data VLAN test IPv4/IPv6 connectivity.  A server on VLAN 13 provides an internal service (e.g. web or file server), and a server in the Internet cloud demonstrates public reachability. |

## Design highlights

### Network segmentation

* **Voice and data separation:**  Each access port is configured with a data VLAN (untagged) and a voice VLAN (tagged) so that IP phones and PCs can coexist on the same physical port.  CDPv2 advertises the configured voice VLAN to legacy phones so that they correctly tag their traffic:contentReference[oaicite:12]{index=12}.  Before CDPv2 can advertise the VLAN ID, the VLAN must be created and flagged as a voice VLAN on the switch:contentReference[oaicite:13]{index=13}.
* **Wireless and management isolation:**  A dedicated VLAN carries CAPWAP traffic between the WLC and the lightweight APs.  An additional management IP (192.168.1.2/24) is used for out‑of‑band access to the WLC.
* **Server and infrastructure networks:**  The internal server resides on VLAN 13 to isolate server traffic from user traffic.  /30 point‑to‑point links (`10.0.255.0/30` and `10.0.255.4/30`) connect the distribution switch to the internal router and firewall respectively.

### Inter‑VLAN routing and EtherChannel

The Catalyst 3560 is configured with Switched Virtual Interfaces (SVIs) for VLANs 10–13.  Each SVI has an IPv4 gateway address and an IPv6 gateway for VLAN 10.  A default route points towards the ASA for Internet‑bound traffic.  The two 2960 access switches form an EtherChannel (LACP) to the 3560, providing increased bandwidth and resiliency.

### Firewall and NAT

All outbound traffic passes through the ASA 5506‑X.  The firewall implements stateful inspection and translates internal addresses to a single public address using **Dynamic Port Address Translation (PAT)**.  In Dynamic PAT the firewall modifies both IP addresses and port numbers so that multiple internal hosts can share one public IP address:contentReference[oaicite:14]{index=14}.  The ASA can also perform **Dynamic NAT**, which modifies only IP addresses and gives each internal host a temporary public address:contentReference[oaicite:15]{index=15}, but Dynamic PAT is more common in small networks.  NAT is configured using network objects and `nat` statements inside the ASA configuration.

### Routing

Router 0 and the 3560 have static routes pointing towards the ASA for default routing.  Router 1 has a static route to the inside (50.1.2.0/30) and a default route towards the “Internet server”.  The 3560 may also run a dynamic routing protocol such as EIGRP or OSPF for practice; however, static routing is sufficient for this lab.

### Wireless LAN deployment

The WLC 2504 centralises management of the lightweight APs.  According to the product features, the controller can support up to **75 access points and 1 000 clients**:contentReference[oaicite:16]{index=16} and offers advanced security and management features:contentReference[oaicite:17]{index=17}.  It is ideal for small to medium environments:contentReference[oaicite:18]{index=18}.  To bring the WLC online you connect it to the switch, browse to its management IP, log in with the `admin` / `Cisco123` credentials and follow the wizard to register the APs:contentReference[oaicite:19]{index=19}.

### Platform capabilities

* **Multilayer switching:**  The Catalyst 3560 provides 802.3af PoE and pre‑standard PoE to power phones and access points.  It delivers enterprise‑class intelligent services such as QoS, rate limiting, ACLs and multicast management, and offers high‑performance IP routing:contentReference[oaicite:20]{index=20}.  It features 24 Fast Ethernet ports with SFP uplinks and includes a Standard Multilayer Image that supports basic RIP and static routing; dynamic routing can be enabled with a software upgrade:contentReference[oaicite:21]{index=21}.
* **Next‑generation firewall:**  The ASA 5506‑X uses a 64‑bit multicore CPU to achieve higher firewall and VPN throughput and can run multiple security services at once:contentReference[oaicite:22]{index=22}.  It integrates threat defence features such as application visibility and control, built‑in IPS, botnet traffic filtering, and integration with Cisco AnyConnect clients:contentReference[oaicite:23]{index=23}.
* **Integrated services routers:**  The Cisco ISR 4321 provides high‑speed connectivity and security features for small and large enterprises:contentReference[oaicite:24]{index=24}.  It can handle heavy traffic without performance loss and protects the network with built‑in intrusion prevention, firewall and VPN capabilities:contentReference[oaicite:25]{index=25}.  The router is also easy to configure thanks to its intuitive interface:contentReference[oaicite:26]{index=26}.

## Configuration summary

The following snippets illustrate how the lab might be configured. The actual Packet Tracer file already contains working
configurations.

### Core0 (3650-24PS Multilayer Switch)

The distribution switch in this lab is a Catalyst 3560 (24‑port PoE model) running IOS XE 16.3.2.  It acts as the default gateway for all internal VLANs, provides DHCP for the wireless management segment and routes traffic towards the firewall and router.  Highlights of the running configuration are summarised below:

* **Global services:**  IP and IPv6 routing are enabled (`ip routing`, `ipv6 unicast‑routing`), LLDP is running for device discovery (`lldp run`) and the spanning‑tree mode is set to Rapid PVST+ with a root priority of 4096 for all VLANs.  IP flow export is enabled to allow NetFlow exports.
* **DHCP:**  A small DHCP pool called `wireless` hands out management addresses in the `192.168.1.0/28` range for the WLC and LAPs.  The address `192.168.1.2` is excluded because it is statically assigned to the WLC.  Other VLANs use DHCP relays (`ip helper‑address`) pointing at an external server, but these are configured within the Packet Tracer file rather than shown here.
* **Layer‑3 links:**  The switch uses routed ports to connect to the internal router and firewall.  GigabitEthernet1/0/3 is configured as a routed interface with the address `10.0.255.1/30` towards Router 0, and GigabitEthernet1/0/5 uses `10.0.255.5/30` towards the ASA.  All other uplinks are configured as trunks or are unused in this lab.
* **SVIs:**  Switched Virtual Interfaces provide gateway addresses for each VLAN.  VLAN 1 (`192.168.1.1/28`) is used for management, VLAN 10 uses `10.0.10.1/24` and IPv6 `2001:10::1/64`, VLAN 11 uses `10.0.11.1/24`, VLAN 12 uses `10.0.12.1/24` and VLAN 13 uses `10.0.13.1/24`.  Each SVI has a unique MAC address specified to simulate different gateways in Packet Tracer.
* **Static route:**  A static route points specific traffic (`1.1.1.1/32`) towards the internal router at `10.0.255.2`.  In a real deployment this would typically be a default route pointing towards the firewall; however, this lab uses a /32 route to demonstrate static routing.

```text
hostname Core0
!
ip dhcp excluded‑address 192.168.1.2
ip dhcp pool wireless
 network 192.168.1.0 255.255.255.240
 default‑router 192.168.1.1
!
ip routing
ipv6 unicast‑routing
lldp run
spanning‑tree mode rapid‑pvst
spanning‑tree vlan 1‑4094 priority 4096
!
interface GigabitEthernet1/0/1
 switchport mode trunk
!
interface GigabitEthernet1/0/2
 switchport mode trunk
!
interface GigabitEthernet1/0/3
 no switchport
 ip address 10.0.255.1 255.255.255.252
!
interface GigabitEthernet1/0/5
 no switchport
 ip address 10.0.255.5 255.255.255.252
!
interface Vlan1
 ip address 192.168.1.1 255.255.255.240
!
interface Vlan10
 mac‑address 00d0.5807.ad01
 ip address 10.0.10.1 255.255.255.0
 ipv6 address 2001:10::1/64
!
interface Vlan11
 mac‑address 00d0.5807.ad02
 ip address 10.0.11.1 255.255.255.0
!
interface Vlan12
 mac‑address 00d0.5807.ad03
 ip address 10.0.12.1 255.255.255.0
!
interface Vlan13
 mac‑address 00d0.5807.ad04
 ip address 10.0.13.1 255.255.255.0
!
ip route 1.1.1.1 255.255.255.255 10.0.255.2

```


### switch0 (2960-24TT access switch)
```text
! Trunk towards distribution switch
interface GigabitEthernet0/1
 description To 3560 (Port‑channel)
 switchport mode trunk
 channel‑group 1 mode active

! Access port for PC and phone
interface FastEthernet0/10
 switchport mode access
 switchport access vlan 10
 switchport voice vlan 11    ! Voice VLAN is advertised via CDPv2:contentReference[oaicite:27]{index=27}
 mls qos trust device cisco‑phone
 spanning‑tree portfast
!
interface FastEthernet0/11
 description Access point
 switchport access vlan 12
 power inline auto
 spanning‑tree portfast
```


### ciscoasa (Cisco ASA 5506‑X)
This configuration positions the ASA as the firewall between the trusted network (`10.0.255.4/30`) and the external network (`50.1.2.0/30`). It assigns security levels, defines inspection rules, and prepares the device for NAT once objects and NAT statements are added.


* **Inside interface (`GigabitEthernet1/1`)** – Named `inside` and set to the highest security level (100). It uses IP `10.0.255.6/30` and connects to the Core0 switch on the `10.0.255.4/30` subnet (Core0 uses `10.0.255.5/30`). All internal VLAN traffic enters the firewall via this link.
* **Outside interface (`GigabitEthernet1/2`)** – Named `outside` with the lowest security level (0). It uses IP `50.1.2.1/30` and connects to the edge router (`Router1`) on the `50.1.2.0/30` subnet. Router1 uses `50.1.2.2/30`, providing the ASA’s path to the Internet or external network.
* **Unused interfaces** – Interfaces `GigabitEthernet1/3–1/8` and `Management1/1` are not needed in this lab. They have no `nameif` assigned, no security level, no IP address and are shut down.
* **Inspection policy** – The `class-map` and `policy-map` entries create a global inspection policy. It matches default inspection traffic and inspects DNS, FTP and TFTP protocols. This allows the ASA to perform stateful inspection on these protocols and enforce protocol‑specific security rules.
* **Service policy** – `service-policy global_policy global` applies the inspection policy to all interfaces.
* **Management time‑outs** – Telnet and SSH sessions time out after five minutes, controlling session duration.
* **NAT/PAT** – NAT rules are not shown here; they would normally translate internal addresses on the `inside` interface to a public IP on the `outside` interface. In the full lab configuration, you would define network objects for your internal subnets (e.g., `10.0.0.0/16`) and add `nat (inside,outside)` commands to implement Dynamic PAT or Static NAT as needed.

```text
hostname ciscoasa

interface GigabitEthernet1/1
 nameif inside
 security-level 100
 ip address 10.0.255.6 255.255.255.252

interface GigabitEthernet1/2
 nameif outside
 security-level 0
 ip address 50.1.2.1 255.255.255.252

interface GigabitEthernet1/3
 no nameif
 no security-level
 no ip address
 shutdown

interface GigabitEthernet1/4
 no nameif
 no security-level
 no ip address
 shutdown

interface GigabitEthernet1/5
 no nameif
 no security-level
 no ip address
 shutdown

interface GigabitEthernet1/6
 no nameif
 no security-level
 no ip address
 shutdown

interface GigabitEthernet1/7
 no nameif
 no security-level
 no ip address
 shutdown

interface GigabitEthernet1/8
 no nameif
 no security-level
 no ip address
 shutdown

interface Management1/1
 management-only
 no nameif
 no security-level
 no ip address
 shutdown

class-map inspection_default
 match default-inspection-traffic

policy-map type inspect dns preset_dns_map
 parameters
  message-length maximum 512

policy-map global_policy
 class inspection_default
  inspect dns preset_dns_map
  inspect ftp
  inspect tftp

service-policy global_policy global

telnet timeout 5
ssh timeout 5
```

### Router (Cisco Router ISR4321)
Router essentially acts as the internal router between the core distribution switch and the ASA firewall. It learns routes via static entries and provides a routed path for packets leaving the VLANs on the core switch toward the edge of the network.

* **Interface GigabitEthernet0/0/0 (10.0.255.2/30)** – This routed link connects Router0 to the Core (Catalyst 3560) switch. The core switch’s corresponding interface is Gig1/0/3 with IP `10.0.255.1/30`. Together these form a point-to-point link on the `10.0.255.0/30` subnet. It serves as the router’s gateway to all internal VLANs.

* **Loopback1 (1.1.1.1/32)** – A loopback interface is often used to represent the router in routing protocols and for testing reachability. In this lab, it also acts as the destination for the static route configured on the core switch (`ip route 1.1.1.1 255.255.255.255 10.0.255.2`).

* **Static route** – `ip route 10.0.10.0 255.255.255.0 10.0.255.1` points traffic destined for the data VLAN (10.0.10.0/24) to the Core switch’s IP (`10.0.255.1`). The core switch then forwards packets to the appropriate VLAN interface. You could add similar routes for other VLANs if needed (e.g., 10.0.11.0/24, 10.0.12.0/24, 10.0.13.0/24).

* **CEF** – Cisco Express Forwarding is enabled for IPv4, improving packet-switching performance. IPv6 CEF is disabled here because this router isn’t handling IPv6 traffic.

* **NetFlow export** – `ip flow-export version 9` prepares the router to export NetFlow records (though it doesn’t specify a destination). This could be used for traffic analysis.


```text
hostname Router

ip cef                         ! Enables Cisco Express Forwarding (CEF) for IPv4
no ipv6 cef                    ! Disables CEF for IPv6 (IPv6 isn’t used on this router)

spanning-tree mode pvst        ! Enables Per‑VLAN Spanning Tree (PVST+) if the router acts as a bridge

! Loopback interface used for testing and as an identifier in routing protocols
interface Loopback1
 ip address 1.1.1.1 255.255.255.255

! Connection to the core switch
interface GigabitEthernet0/0/0
 ip address 10.0.255.2 255.255.255.252
 duplex auto
 speed auto

! Unused interface (shut by default)
interface GigabitEthernet0/0/1
 no ip address
 duplex auto
 speed auto

! Default VLAN interface (not used, shut down)
interface Vlan1
 no ip address
 shutdown

ip classless                   ! Allows classless routing (routes to non-classful networks)

! Static route to reach the data VLAN (10.0.10.0/24) via the Core switch
ip route 10.0.10.0 255.255.255.0 10.0.255.1

ip flow-export version 9       ! Sets the NetFlow export version

line con 0
line aux 0
line vty 0 4
 login
```

### Wireless LAN Controller (WLC-2504)

The WLC provides centralised management for all lightweight access points in the lab. Key details of the controller’s are summarised below:

- **Management IP address:** `192.168.1.2` (on the management VLAN). This is the address used to reach the WLC via HTTPS, with `192.168.1.1` on Core0 acting as the default gateway.
- **Radio states:** Both the **802.11a/n/ac** (5 GHz) and **802.11b/g/n** (2.4 GHz) networks are **enabled**, so the APs broadcast on both bands.
- **Access point summary:** Two lightweight APs are registered; each has its 5 GHz and 2.4 GHz radios **up** (0 down). This matches the two LAPs connected in the lab topology.
- **Client summary:** The WLC reports **2 current clients**, which aligns with the smartphones connected to the wireless VLAN.
- **Rogue detection:** The **Rogue Summary** shows zero rogue APs, clients, or ad‑hoc rogues, indicating no unauthorized wireless activity has been detected.
- **System utilisation:** CPU utilisation is negligible (0 % across cores), memory usage is around **46 %**, and fan speed is **3800 rpm**, which are typical for a lightly loaded controller.

#### Dashboard screenshot

To provide a visual reference of the dashboard screenshot below:
<img width="2781" height="1301" alt="image" src="https://github.com/user-attachments/assets/99543a0c-864a-4a53-9743-8cdf7f4359aa" />
<img width="2827" height="672" alt="image" src="https://github.com/user-attachments/assets/6acf520c-3152-43d6-9e66-05716007a749" />

