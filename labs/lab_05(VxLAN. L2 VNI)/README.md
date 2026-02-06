# Лабораторная работа:  Настройка Overlay на основе VxLAN EVPN для L2 связанности между клиентами

## Задание
1. Настроить Overlay на основе VxLAN EVPN для обеспечения L2-связности между всеми сетевыми устройствами
2. Задокументировать:
   - Схему сети
   - Конфигурацию устройств
3. Проверить IP-связность между устройствами в BGP домене

---

## Топология сети
![Схема Сети ](fabric_scheme.jpg)
## IP-план (Address Plan)

### Underlay сеть (Fabric Links - Point-to-Point /31)

| Device Name | IP Address/Маска | Port      | Remote Device | Remote Port | Description         |
|-------------|------------------|-----------|---------------|-------------|---------------------|
| 99-blf1     | 10.99.241.0/31   | Ethernet1 | 99-sp1        | Ethernet1   | to Spine1           |
| 99-blf1     | 10.99.242.0/31   | Ethernet2 | 99-sp2        | Ethernet1   | to Spine2           |
| 99-blf1     | -                | Ethernet3 | 99-esx1       | Ethernet1   | Server Trunk        |
| 99-blf1     | -                | Ethernet4 | 99-esx4       | Ethernet2   | Server Trunk        |
| 99-blf2     | 10.99.241.2/31   | Ethernet1 | 99-sp1        | Ethernet2   | to Spine1           |
| 99-blf2     | 10.99.242.2/31   | Ethernet2 | 99-sp2        | Ethernet2   | to Spine2           |
| 99-blf2     | -                | Ethernet3 | 99-esx2       | Ethernet1   | Server Trunk        |
| 99-blf2     | -                | Ethernet4 | 99-esx3       | Ethernet2   | Server Trunk        |
| 99-lf3      | 10.99.241.4/31   | Ethernet1 | 99-sp1        | Ethernet3   | to Spine1           |
| 99-lf3      | 10.99.242.4/31   | Ethernet2 | 99-sp2        | Ethernet3   | to Spine2           |
| 99-lf3      | -                | Ethernet3 | 99-esx3       | Ethernet1   | Server Trunk        |
| 99-lf3      | -                | Ethernet4 | 99-esx2       | Ethernet2   | Server Trunk        |
| 99-lf4      | 10.99.241.6/31   | Ethernet1 | 99-sp1        | Ethernet4   | to Spine1           |
| 99-lf4      | 10.99.242.6/31   | Ethernet2 | 99-sp2        | Ethernet4   | to Spine2           |
| 99-lf4      | -                | Ethernet3 | 99-esx4       | Ethernet1   | Server Trunk        |
| 99-lf4      | -                | Ethernet4 | 99-esx1       | Ethernet2   | Server Trunk        |

### Loopback адреса для BGP Underlay (Сеть 10.99.243.0/24)

| Device Name | Loopback0 Address | Description          |
|-------------|-------------------|----------------------|
| 99-blf1     | 10.99.243.1/32    | BGP Router-ID        |
| 99-blf2     | 10.99.243.2/32    | BGP Router-ID        |
| 99-lf3      | 10.99.243.3/32    | BGP Router-ID        |
| 99-lf4      | 10.99.243.4/32    | BGP Router-ID        |
| 99-sp1      | 10.99.243.11/32   | BGP Router-ID        |
| 99-sp2      | 10.99.243.22/32   | BGP Router-ID        |

### Loopback адреса для VXLAN NVE (Сеть 10.99.244.0/24)

| Device Name | Loopback1 Address | Description          |
|-------------|-------------------|----------------------|
| 99-blf1     | 10.99.244.1/32    | VTEP Source          |
| 99-blf2     | 10.99.244.2/32    | VTEP Source          |
| 99-lf3      | 10.99.244.3/32    | VTEP Source          |
| 99-lf4      | 10.99.244.4/32    | VTEP Source          |

### EVPN L2-домены (VLAN ↔ VNI Mapping)

| VLAN | Описание          | L2 VNI | Anycast Gateway   | Leaf с настроенным VNI |
|------|-------------------|--------|-------------------|------------------------|
| 10   | VLAN10   | 10010  | 192.168.1.254/24  | 99-blf1, 99-blf2, 99-lf3, 99-lf4 |
| 20   | VLAN20   | 10020  | 192.168.2.254/24  | 99-blf1, 99-blf2, 99-lf3, 99-lf4 |

### Конфигурация "ESXi" устройств 

| ESXi Device | Физические порты (Trunk) | Виртуальные интерфейсы (SVI)| IP Address/Маска   |
|-------------|--------------------------|-----------------------------|--------------------|
| 99-esx1     | Eth1 → 99-blf1 Eth3      | Vlan10 (SVI для VLAN10)     |  192.168.1.1/24    |
|             | Eth1 → 99-blf1 Eth3      | Vlan20 (SVI для VLAN20)     |  192.168.2.1/24    |
| 99-esx2     | Eth1 → 99-blf2 Eth3      | Vlan10 (SVI для VLAN10)     |  192.168.1.2/24    |
|             | Eth1 → 99-blf2 Eth3      | Vlan20 (SVI для VLAN20)     |  192.168.2.2/24    |
| 99-esx3     | Eth1 → 99-lf3 Eth3       | Vlan10 (SVI для VLAN10)     |  192.168.1.3/24    |
|             | Eth1 → 99-lf3 Eth3      | Vlan20 (SVI для VLAN20)      |  192.168.2.3/24    |
| 99-esx4     | Eth1 → 99-lf4 Eth3       | Vlan10 (SVI для VLAN10)     |  192.168.1.4/24    |
|             | Eth1 → 99-lf4 Eth3     | Vlan20 (SVI для VLAN20)       |  192.168.2.4/24    | 

---

## Конфигурация BGP1
### 99-blf1 (Border Leaf 1)
```bash
configure terminal
hostname 99-blf1

ip routing
service routing protocols model multi-agent

vlan 10
   name VLAN10

vlan 20
   name VLAN20

interface Ethernet1
   description to-99-sp1-Eth1
   no switchport
   mtu 9100
   ip address 10.99.241.0/31
   

interface Ethernet2
   description to-99-sp2-Eth1
   no switchport
   mtu 9100
   ip address 10.99.242.0/31
   

interface Ethernet3
   description to-99-esx1-Eth1
   switchport mode trunk
   switchport trunk allowed vlan 10,20
   mtu 9100
   no shutdown


interface Vlan10
   description Vlan10
   mtu 9100
   ip address 192.168.1.251/24

interface Vlan20
   description Vlan20
   mtu 9100
   ip address 192.168.2.251/24

interface Loopback0
   description Router-ID
   ip address 10.99.243.1/32

interface Loopback1
   description VTEP-Source
   ip address 10.99.244.1/32

interface Vxlan1
   description VXLAN-Tunnel-Endpoint
   no shutdown
   vxlan source-interface Loopback1
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
   
  
router bgp 65099
   router-id 10.99.243.1
   no bgp default ipv4-unicast
   timers bgp 3 9
   maximum-paths 10 ecmp 10

   neighbor SPINE-UNDERLAY peer group
   neighbor SPINE-UNDERLAY remote-as 65099
   neighbor SPINE-UNDERLAY timers 3 9
   neighbor SPINE-UNDERLAY send-community
   neighbor SPINE-UNDERLAY bfd interval 300 min-rx 300 multiplier 3
   neighbor 10.99.241.1 peer group SPINE-UNDERLAY
   neighbor 10.99.242.1 peer group SPINE-UNDERLAY

   neighbor SPINE-EVPN peer group
   neighbor SPINE-EVPN remote-as 65099
   neighbor SPINE-EVPN update-source Loopback0
   neighbor SPINE-EVPN ebgp-multihop 2
   neighbor SPINE-EVPN send-community extended
   neighbor 10.99.243.11 peer group SPINE-EVPN
   neighbor 10.99.243.22 peer group SPINE-EVPN
   vlan 10 
   rd auto 
   route-target both 65099:10010
   redistribute learned
   vlan 20 
   rd auto 
   route-target both 65099:10020
   redistribute learned
   
   address-family ipv4
      neighbor SPINE-UNDERLAY activate
      network 10.99.243.1/32
      network 10.99.244.1/32

   
   address-family evpn
      neighbor SPINE-EVPN activate


 ```

 ### 99-blf2 (Border Leaf 2)
 ```bash
 configure terminal
hostname 99-blf2

ip routing
service routing protocols model multi-agent

vlan 10
   name VLAN10

vlan 20
   name VLAN20

interface Ethernet1
   description to-99-sp1-Eth2
   no switchport
   mtu 9100
   ip address 10.99.241.2/31
   

interface Ethernet2
   description to-99-sp2-Eth2
   no switchport
   mtu 9100
   ip address 10.99.242.2/31
   

interface Ethernet3
   description to-99-esx2-Eth1
   switchport mode trunk
   switchport trunk allowed vlan 10,20
   mtu 9100
   no shutdown


interface Vlan10
   description Vlan10
   mtu 9100
   ip address 192.168.1.252/24

interface Vlan20
   description Vlan20
   mtu 9100
   ip address 192.168.2.252/24

interface Loopback0
   description Router-ID
   ip address 10.99.243.2/32

interface Loopback1
   description VTEP-Source
   ip address 10.99.244.2/32

interface Vxlan1
   description VXLAN-Tunnel-Endpoint
   no shutdown
   vxlan source-interface Loopback1
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
  
router bgp 65099
   router-id 10.99.243.2
   no bgp default ipv4-unicast
   timers bgp 3 9
   maximum-paths 10 ecmp 10

   neighbor SPINE-UNDERLAY peer group
   neighbor SPINE-UNDERLAY remote-as 65099
   neighbor SPINE-UNDERLAY timers 3 9
   neighbor SPINE-UNDERLAY send-community
   neighbor SPINE-UNDERLAY bfd interval 300 min-rx 300 multiplier 3
   neighbor 10.99.241.3 peer group SPINE-UNDERLAY
   neighbor 10.99.242.3 peer group SPINE-UNDERLAY

   neighbor SPINE-EVPN peer group
   neighbor SPINE-EVPN remote-as 65099
   neighbor SPINE-EVPN update-source Loopback0
   neighbor SPINE-EVPN ebgp-multihop 2
   neighbor SPINE-EVPN send-community extended
   neighbor 10.99.243.11 peer group SPINE-EVPN
   neighbor 10.99.243.22 peer group SPINE-EVPN
   
   vlan 10 
   rd auto 
   route-target both 65099:10010
   redistribute learned
   
   vlan 20 
   rd auto 
   route-target both 65099:10020
   redistribute learned
   
   address-family ipv4
      neighbor SPINE-UNDERLAY activate
      network 10.99.243.2/32
      network 10.99.244.2/32

   
   address-family evpn
      neighbor SPINE-EVPN activate

 ```
### 99-lf3 (Leaf 3)
```bash
configure terminal
hostname 99-lf3

ip routing
service routing protocols model multi-agent

vlan 10
   name VLAN10

vlan 20
   name VLAN20

interface Ethernet1
   description to-99-sp1-Eth3
   no switchport
   mtu 9100
   ip address 10.99.241.4/31
   

interface Ethernet2
   description to-99-sp2-Eth3
   no switchport
   mtu 9100
   

interface Ethernet3
   description to-99-esx3-Eth1
   switchport mode trunk
   switchport trunk allowed vlan 10,20
   mtu 9100
   no shutdown


interface Vlan10
   description Vlan10
   mtu 9100
   ip address 192.168.1.253/24

interface Vlan20
   description Vlan20
   mtu 9100
   ip address 192.168.2.253/24

interface Loopback0
   description Router-ID
   ip address 10.99.243.3/32

interface Loopback1
   description VTEP-Source
   ip address 10.99.244.3/32

interface Vxlan1
   description VXLAN-Tunnel-Endpoint
   no shutdown
   vxlan source-interface Loopback1
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
  
router bgp 65099
   router-id 10.99.243.3
   no bgp default ipv4-unicast
   timers bgp 3 9
   maximum-paths 10 ecmp 10

   neighbor SPINE-UNDERLAY peer group
   neighbor SPINE-UNDERLAY remote-as 65099
   neighbor SPINE-UNDERLAY timers 3 9
   neighbor SPINE-UNDERLAY send-community
   neighbor SPINE-UNDERLAY bfd interval 300 min-rx 300 multiplier 3
   neighbor 10.99.241.5 peer group SPINE-UNDERLAY
   neighbor 10.99.242.5 peer group SPINE-UNDERLAY

   neighbor SPINE-EVPN peer group
   neighbor SPINE-EVPN remote-as 65099
   neighbor SPINE-EVPN update-source Loopback0
   neighbor SPINE-EVPN ebgp-multihop 2
   neighbor SPINE-EVPN send-community extended
   neighbor 10.99.243.11 peer group SPINE-EVPN
   neighbor 10.99.243.22 peer group SPINE-EVPN
   
   vlan 10 
   rd auto 
   route-target both 65099:10010
   redistribute learned
   
   vlan 20 
   rd auto 
   route-target both 65099:10020
   redistribute learned
   
   address-family ipv4
      neighbor SPINE-UNDERLAY activate
      network 10.99.243.3/32
      network 10.99.244.3/32

   
   address-family evpn
      neighbor SPINE-EVPN activate
 ```
 ### 99-lf4 (Leaf 4)
```bash
configure terminal
hostname 99-lf4

ip routing
service routing protocols model multi-agent

vlan 10
   name VLAN10

vlan 20
   name VLAN20

interface Ethernet1
   description to-99-sp1-Eth4
   no switchport
   mtu 9100
   ip address 10.99.241.6/31
   

interface Ethernet2
   description to-99-sp2-Eth4
   no switchport
   mtu 9100
   ip address 10.99.242.6/31
   

interface Ethernet3
   description to-99-esx4-Eth1
   switchport mode trunk
   switchport trunk allowed vlan 10,20
   mtu 9100
   no shutdown

interface Vlan10
   description Vlan10
   mtu 9100
   ip address 192.168.1.254/24

interface Vlan20
   description Vlan20
   mtu 9100
   ip address 192.168.2.254/24

interface Loopback0
   description Router-ID
   ip address 10.99.243.4/32

interface Loopback1
   description VTEP-Source
   ip address 10.99.244.4/32

interface Vxlan1
   description VXLAN-Tunnel-Endpoint
   no shutdown
   vxlan source-interface Loopback1
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
   
router bgp 65099
   router-id 10.99.243.4
   no bgp default ipv4-unicast
   timers bgp 3 9
   maximum-paths 10 ecmp 10

   neighbor SPINE-UNDERLAY peer group
   neighbor SPINE-UNDERLAY remote-as 65099
   neighbor SPINE-UNDERLAY timers 3 9
   neighbor SPINE-UNDERLAY send-community
   neighbor SPINE-UNDERLAY bfd interval 300 min-rx 300 multiplier 3
   neighbor 10.99.241.7 peer group SPINE-UNDERLAY
   neighbor 10.99.242.7 peer group SPINE-UNDERLAY

   neighbor SPINE-EVPN peer group
   neighbor SPINE-EVPN remote-as 65099
   neighbor SPINE-EVPN update-source Loopback0
   neighbor SPINE-EVPN ebgp-multihop 2
   neighbor SPINE-EVPN send-community extended
   neighbor 10.99.243.11 peer group SPINE-EVPN
   neighbor 10.99.243.22 peer group SPINE-EVPN
   
   vlan 10 
   rd auto 
   route-target both 65099:10010
   redistribute learned
   
   vlan 20 
   rd auto 
   route-target both 65099:10020
   redistribute learned
   
   address-family ipv4
      neighbor SPINE-UNDERLAY activate
      network 10.99.243.4/32
      network 10.99.244.4/32

   
   address-family evpn
      neighbor SPINE-EVPN activate
 ```

 ### 99-sp1 (Spine 1)
 ```bash
configure terminal
hostname 99-sp1

ip routing
service routing protocols model multi-agent

interface Ethernet1
   description to-99-blf1-Eth1
   no switchport
   mtu 9100
   ip address 10.99.241.1/31
   

interface Ethernet2
   description to-99-blf2-Eth1
   no switchport
   mtu 9100
   ip address 10.99.241.3/31
   

interface Ethernet3
   description to-99-lf3-Eth1
   no switchport
   mtu 9100
   ip address 10.99.241.5/31
   

interface Ethernet4
   description to-99-lf4-Eth1
   no switchport
   mtu 9100
   ip address 10.99.241.7/31
   

interface Loopback0
   description Router-ID
   ip address 10.99.243.11/32

router bgp 65099
   router-id 10.99.243.11
   no bgp default ipv4-unicast
   timers bgp 3 9
   maximum-paths 10 ecmp 10
   
   neighbor LEAF-UNDERLAY peer group
   neighbor LEAF-UNDERLAY remote-as 65099
   neighbor LEAF-UNDERLAY route-reflector-client
   neighbor LEAF-UNDERLAY send-community
   neighbor LEAF-UNDERLAY next-hop-self
   neighbor LEAF-UNDERLAY interval 300 min-rx 300 multiplier 3
   bgp listen range 10.99.241.0/24 peer-group LEAF-UNDERLAY remote-as 65099
   
   neighbor LEAF-EVPN peer group
   neighbor LEAF-EVPN remote-as 65099
   neighbor LEAF-EVPN update-source Loopback0
   neighbor LEAF-EVPN route-reflector-client
   neighbor LEAF-EVPN send-community extended
   neighbor 10.99.243.1 peer group LEAF-EVPN
   neighbor 10.99.243.2 peer group LEAF-EVPN
   neighbor 10.99.243.3 peer group LEAF-EVPN
   neighbor 10.99.243.4 peer group LEAF-EVPN
   
   address-family ipv4
      neighbor LEAF-UNDERLAY activate
      network 10.99.243.11/32
   
   address-family evpn
      neighbor LEAF-EVPN activate
 ```

 ### 99-sp2 (Spine 2)
 ```bash
configure terminal
hostname 99-sp2

ip routing
service routing protocols model multi-agent

interface Ethernet1
   description to-99-blf1-Eth2
   no switchport
   mtu 9100
   ip address 10.99.242.1/31
   

interface Ethernet2
   description to-99-blf2-Eth2
   no switchport
   mtu 9100
   ip address 10.99.242.3/31
   

interface Ethernet3
   description to-99-lf3-Eth2
   no switchport
   mtu 9100
   ip address 10.99.242.5/31
   

interface Ethernet4
   description to-99-lf4-Eth2
   no switchport
   mtu 9100
   ip address 10.99.242.7/31
   

interface Loopback0
   description Router-ID
   ip address 10.99.243.22/32

router bgp 65099
   router-id 10.99.243.22
   no bgp default ipv4-unicast
   timers bgp 3 9
   maximum-paths 10 ecmp 10
   
   neighbor LEAF-UNDERLAY peer group
   neighbor LEAF-UNDERLAY remote-as 65099
   neighbor LEAF-UNDERLAY route-reflector-client
   neighbor LEAF-UNDERLAY send-community
   neighbor LEAF-UNDERLAY next-hop-self
   neighbor LEAF-UNDERLAY bfd interval 300 min-rx 300 multiplier 3
   bgp listen range 10.99.241.0/24 peer-group LEAF-UNDERLAY remote-as 65099
   
   neighbor LEAF-EVPN peer group
   neighbor LEAF-EVPN remote-as 65099
   neighbor LEAF-EVPN update-source Loopback0
   neighbor LEAF-EVPN route-reflector-client
   neighbor LEAF-EVPN send-community extended
   neighbor 10.99.243.1 peer group LEAF-EVPN
   neighbor 10.99.243.2 peer group LEAF-EVPN
   neighbor 10.99.243.3 peer group LEAF-EVPN
   neighbor 10.99.243.4 peer group LEAF-EVPN
   
   address-family ipv4
      neighbor LEAF-UNDERLAY activate
      network 10.99.243.22/32
   
   address-family evpn
      neighbor LEAF-EVPN activate

 ```
 

  ### 99-esxN (ESX N)
 ```bash
configure terminal
hostname 99-esxN

ip routing
vlan 10
vlan 20
interface Ethernet1
   description to-99-lf4-Eth3
   switchport mode trunk
   switchport trunk allowed vlan 10,20
   mtu 9100
   no shutdown

interface Ethernet2
   description to-99-lf3-Eth4
   switchport mode trunk
   switchport trunk allowed vlan 10,20
   mtu 9100
   no shutdown

interface Vlan10
   description VMKernel-VLAN10
   ip address 192.168.1.N/24
   mtu 9100
   no shutdown

interface Vlan20
   description VMKernel-VLAN20
   ip address 192.168.2.N/24
   mtu 9100
   no shutdown



 ```
---

## Проверка IP связности
 ## 1. Проверка VXLAN соседств
```
99-lf4#sh vxlan vtep
Remote VTEPS for Vxlan1:

VTEP              Tunnel Type(s)
----------------- --------------
10.99.244.1       flood
10.99.244.2       flood
10.99.244.3       flood

Total number of remote VTEPS:  3

```
```
99-lf4#sh int vxlan 1
Vxlan1 is up, line protocol is up (connected)
  Hardware is Vxlan
  Description: VXLAN-Tunnel-Endpoint
  Source interface is Loopback1 and is active with 10.99.244.4
  Listening on UDP port 4789
  Replication/Flood Mode is headend with Flood List Source: EVPN
  Remote MAC learning via EVPN
  VNI mapping to VLANs
  Static VLAN to VNI mapping is
    [10, 10010]       [20, 10020]
  Note: All Dynamic VLANs used by VCS are internal VLANs.
        Use 'show vxlan vni' for details.
  Static VRF to VNI mapping is not configured
  Headend replication flood vtep list is:
    10 10.99.244.3     10.99.244.1     10.99.244.2
    20 10.99.244.3     10.99.244.1     10.99.244.2
  Shared Router MAC is 0000.0000.0000
```
```
99-lf4#sh vxlan vni
VNI to VLAN Mapping for Vxlan1
VNI         VLAN       Source       Interface       802.1Q Tag
----------- ---------- ------------ --------------- ----------
10010       10         static       Ethernet3       10
                                    Ethernet4       10
                                    Vxlan1          10
10020       20         static       Ethernet3       20
                                    Ethernet4       20
                                    Vxlan1          20

```
```
99-sp2(config-router-bgp)#sh bgp summary
BGP summary information for VRF default
Router identifier 10.99.243.22, local AS number 65099
Neighbor             AS Session State AFI/SAFI                AFI/SAFI State   NLRI Rcd   NLRI Acc
----------- ----------- ------------- ----------------------- -------------- ---------- ----------
10.99.242.0       65099 Established   IPv4 Unicast            Negotiated              2          2
10.99.242.2       65099 Established   IPv4 Unicast            Negotiated              2          2
10.99.242.4       65099 Established   IPv4 Unicast            Negotiated              2          2
10.99.242.6       65099 Established   IPv4 Unicast            Negotiated              2          2
10.99.243.1       65099 Established   L2VPN EVPN              Negotiated              2          2
10.99.243.2       65099 Established   L2VPN EVPN              Negotiated              2          2
10.99.243.3       65099 Established   L2VPN EVPN              Negotiated              2          2
10.99.243.4       65099 Established   L2VPN EVPN              Negotiated              2          2

```

## 2.  Проверка таблицы роутов evpn  
```
99-lf4#sh bgp evpn
BGP routing table information for VRF default
Router identifier 10.99.243.4, local AS number 65099
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.99.243.1:10 imet 10.99.244.1
                                 10.99.244.1           -       100     0       i Or-ID: 10.99.243.1 C-LST: 10.99.243.22
 *  ec    RD: 10.99.243.1:10 imet 10.99.244.1
                                 10.99.244.1           -       100     0       i Or-ID: 10.99.243.1 C-LST: 10.99.243.11
 * >Ec    RD: 10.99.243.1:20 imet 10.99.244.1
                                 10.99.244.1           -       100     0       i Or-ID: 10.99.243.1 C-LST: 10.99.243.22
 *  ec    RD: 10.99.243.1:20 imet 10.99.244.1
                                 10.99.244.1           -       100     0       i Or-ID: 10.99.243.1 C-LST: 10.99.243.11
 * >Ec    RD: 10.99.243.2:10 imet 10.99.244.2
                                 10.99.244.2           -       100     0       i Or-ID: 10.99.243.2 C-LST: 10.99.243.11
 *  ec    RD: 10.99.243.2:10 imet 10.99.244.2
                                 10.99.244.2           -       100     0       i Or-ID: 10.99.243.2 C-LST: 10.99.243.22
 * >Ec    RD: 10.99.243.2:20 imet 10.99.244.2
                                 10.99.244.2           -       100     0       i Or-ID: 10.99.243.2 C-LST: 10.99.243.11
 *  ec    RD: 10.99.243.2:20 imet 10.99.244.2
                                 10.99.244.2           -       100     0       i Or-ID: 10.99.243.2 C-LST: 10.99.243.22
 * >Ec    RD: 10.99.243.3:10 imet 10.99.244.3
                                 10.99.244.3           -       100     0       i Or-ID: 10.99.243.3 C-LST: 10.99.243.22
 *  ec    RD: 10.99.243.3:10 imet 10.99.244.3
                                 10.99.244.3           -       100     0       i Or-ID: 10.99.243.3 C-LST: 10.99.243.11
 * >Ec    RD: 10.99.243.3:20 imet 10.99.244.3
                                 10.99.244.3           -       100     0       i Or-ID: 10.99.243.3 C-LST: 10.99.243.22
 *  ec    RD: 10.99.243.3:20 imet 10.99.244.3
                                 10.99.244.3           -       100     0       i Or-ID: 10.99.243.3 C-LST: 10.99.243.11
 * >      RD: 10.99.243.4:10 imet 10.99.244.4
                                 -                     -       -       0       i
 * >      RD: 10.99.243.4:20 imet 10.99.244.4
                                 -                     -       -       0       i

```
Видим только маршруты тайп 3 
Пингуем с esx1 все айпи в сети 192.168.1.0/24

Видим что приехали маршруты тайп 2

```
99-lf4#sh bgp evpn vni 10010
BGP routing table information for VRF default
Router identifier 10.99.243.4, local AS number 65099
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.99.243.4:10 mac-ip 5000.0003.3766
                                 -                     -       -       0       i
 * >      RD: 10.99.243.4:10 mac-ip 5000.0003.3766 192.168.1.4
                                 -                     -       -       0       i
 * >Ec    RD: 10.99.243.3:10 mac-ip 5000.001b.5e8d
                                 10.99.244.3           -       100     0       i Or-ID: 10.99.243.3 C-LST: 10.99.243.11
 *  ec    RD: 10.99.243.3:10 mac-ip 5000.001b.5e8d
                                 10.99.244.3           -       100     0       i Or-ID: 10.99.243.3 C-LST: 10.99.243.22
 * >Ec    RD: 10.99.243.2:10 mac-ip 5000.006b.2e70
                                 10.99.244.2           -       100     0       i Or-ID: 10.99.243.2 C-LST: 10.99.243.11
 *  ec    RD: 10.99.243.2:10 mac-ip 5000.006b.2e70
                                 10.99.244.2           -       100     0       i Or-ID: 10.99.243.2 C-LST: 10.99.243.22
 * >Ec    RD: 10.99.243.1:10 mac-ip 5000.00d5.5dc0
                                 10.99.244.1           -       100     0       i Or-ID: 10.99.243.1 C-LST: 10.99.243.22
 *  ec    RD: 10.99.243.1:10 mac-ip 5000.00d5.5dc0
                                 10.99.244.1           -       100     0       i Or-ID: 10.99.243.1 C-LST: 10.99.243.11
 * >Ec    RD: 10.99.243.1:10 mac-ip 5000.00d5.5dc0 192.168.1.1
                                 10.99.244.1           -       100     0       i Or-ID: 10.99.243.1 C-LST: 10.99.243.22
 *  ec    RD: 10.99.243.1:10 mac-ip 5000.00d5.5dc0 192.168.1.1
                                 10.99.244.1           -       100     0       i Or-ID: 10.99.243.1 C-LST: 10.99.243.11
 * >Ec    RD: 10.99.243.1:10 imet 10.99.244.1
                                 10.99.244.1           -       100     0       i Or-ID: 10.99.243.1 C-LST: 10.99.243.22
 *  ec    RD: 10.99.243.1:10 imet 10.99.244.1
                                 10.99.244.1           -       100     0       i Or-ID: 10.99.243.1 C-LST: 10.99.243.11
 * >Ec    RD: 10.99.243.2:10 imet 10.99.244.2
                                 10.99.244.2           -       100     0       i Or-ID: 10.99.243.2 C-LST: 10.99.243.11
 *  ec    RD: 10.99.243.2:10 imet 10.99.244.2
                                 10.99.244.2           -       100     0       i Or-ID: 10.99.243.2 C-LST: 10.99.243.22
 * >Ec    RD: 10.99.243.3:10 imet 10.99.244.3
                                 10.99.244.3           -       100     0       i Or-ID: 10.99.243.3 C-LST: 10.99.243.22
 *  ec    RD: 10.99.243.3:10 imet 10.99.244.3
                                 10.99.244.3           -       100     0       i Or-ID: 10.99.243.3 C-LST: 10.99.243.11
 * >      RD: 10.99.243.4:10 imet 10.99.244.4

```


## 3. Проверка межсерверной связности между VM и подсетями 192.168.1.0/24 и 192.158.2.0/24 
```
99-esx4#ping 192.168.1.253 size 8800 df-bit
PING 192.168.1.253 (192.168.1.253) 8772(8800) bytes of data.
8780 bytes from 192.168.1.253: icmp_seq=1 ttl=64 time=56.6 ms
8780 bytes from 192.168.1.253: icmp_seq=2 ttl=64 time=46.4 ms
8780 bytes from 192.168.1.253: icmp_seq=3 ttl=64 time=40.0 ms
8780 bytes from 192.168.1.253: icmp_seq=4 ttl=64 time=34.1 ms
8780 bytes from 192.168.1.253: icmp_seq=5 ttl=64 time=27.2 ms

--- 192.168.1.253 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 44ms
rtt min/avg/max/mdev = 27.225/40.905/56.628/10.116 ms, pipe 5, ipg/ewma 11.220/48.057 ms
99-esx4#
99-esx4#sh ip arp
Address         Age (sec)  Hardware Addr   Interface
192.168.1.1       1:47:59  5000.00d5.5dc0  Vlan10, Ethernet1
192.168.1.3       0:01:24  5000.001b.5e8d  Vlan10, Ethernet1
192.168.1.251     1:57:35  5000.00d7.ee0b  Vlan10, Ethernet1
192.168.1.252     0:00:59  5000.00cb.38c2  Vlan10, Ethernet1
192.168.1.253     0:00:57  5000.00f6.ad37  Vlan10, Ethernet1
192.168.1.254     2:55:12  5000.00af.d3f6  Vlan10, not learned
192.168.2.1       1:43:11  5000.00d5.5dc0  Vlan20, Ethernet1
192.168.2.2       0:01:31  5000.006b.2e70  Vlan20, Ethernet1
192.168.2.3       0:01:27  5000.001b.5e8d  Vlan20, Ethernet1
192.168.2.254     2:26:23  5000.00af.d3f6  Vlan20, not learned


99-esx1#ping 192.168.1.4
PING 192.168.1.4 (192.168.1.4) 72(100) bytes of data.
80 bytes from 192.168.1.4: icmp_seq=1 ttl=64 time=63.1 ms
80 bytes from 192.168.1.4: icmp_seq=2 ttl=64 time=52.9 ms
80 bytes from 192.168.1.4: icmp_seq=3 ttl=64 time=48.6 ms
80 bytes from 192.168.1.4: icmp_seq=5 ttl=64 time=32.9 ms

--- 192.168.1.4 ping statistics ---
5 packets transmitted, 4 received, 20% packet loss, time 43ms
rtt min/avg/max/mdev = 32.927/49.445/63.171/10.890 ms, pipe 5, ipg/ewma 10.956/56.832 ms
99-esx1# 
 
```



