# Лабораторная работа:  Настройка Underlay сети с использованием BGP

## Задание
1. Настроить IBGP в Underlay сети для обеспечения IP-связности между всеми сетевыми устройствами
2. Задокументировать:
   - Схему сети
   - Конфигурацию устройств
3. Проверить IP-связность между устройствами в BGP домене

---

## Топология сети
![Схема Сети ](fabric_scheme.jpg)
## IP-план (Address Plan)

### Underlay сеть (Fabric Links - Point-to-Point /31)
| Device Name | IP Address/Маска | Port | Remote Device | Remote Port | Description |
|-------------|------------------|------|---------------|-------------|-------------|
| 99-blf1 | 10.99.241.0/31 | Ethernet1 | 99-sp1 | Ethernet1 | to Spine1 |
| 99-blf1 | 10.99.242.0/31 | Ethernet2 | 99-sp2 | Ethernet1 | to Spine2 |
| 99-blf1 | 192.168.1.254/24 | Ethernet3 | esx1 | Eth0 | Server Network1 |
| 99-blf2 | 10.99.241.2/31 | Ethernet1 | 99-sp1 | Ethernet2 | to Spine1 |
| 99-blf2 | 10.99.242.2/31 | Ethernet2 | 99-sp2 | Ethernet2 | to Spine2 |
| 99-blf2 | 192.168.2.254/24 | Ethernet3 | esx2 | Eth0 | Server Network2 |
| 99-lf3 | 10.99.241.4/31 | Ethernet1 | 99-sp1 | Ethernet3 | to Spine1 |
| 99-lf3 | 10.99.242.4/31 | Ethernet2 | 99-sp2 | Ethernet3 | to Spine2 |
| 99-lf3 | 192.168.3.254/24 | Ethernet3 | esx3 | Eth0 | Server Network3 |
| 99-lf4 | 10.99.241.6/31 | Ethernet1 | 99-sp1 | Ethernet4 | to Spine1 |
| 99-lf4 | 10.99.242.6/31 | Ethernet2 | 99-sp2 | Ethernet4 | to Spine2 |
| 99-lf4 | 192.168.4.254/24 | Ethernet4 | esx4 | Eth0 | Server Network4 |
| 99-sp1 | 10.99.241.1/31 | Ethernet1 | 99-blf1 | Ethernet1 | to BorderLeaf1 |
| 99-sp1 | 10.99.241.3/31 | Ethernet2 | 99-blf2 | Ethernet1 | to BorderLeaf2 |
| 99-sp1 | 10.99.241.5/31 | Ethernet3 | 99-lf3 | Ethernet1 | to Leaf3 |
| 99-sp1 | 10.99.241.7/31 | Ethernet4 | 99-lf4 | Ethernet1 | to Leaf4 |
| 99-sp2 | 10.99.242.1/31 | Ethernet1 | 99-blf1 | Ethernet2 | to BorderLeaf1 |
| 99-sp2 | 10.99.242.3/31 | Ethernet2 | 99-blf2 | Ethernet2 | to BorderLeaf2 |
| 99-sp2 | 10.99.242.5/31 | Ethernet3 | 99-lf3 | Ethernet2 | to Leaf3 |
| 99-sp2 | 10.99.242.7/31 | Ethernet4 | 99-lf4 | Ethernet2 | to Leaf4 |

### Серверные ВМ
| Device Name | IP Address/Маска | Port | Gateway | Description |
|-------------|------------------|------|---------|-------------|
| esx1 | 192.168.1.2/24 | Eth0 | 192.168.1.254 | VM1 |
| esx2 | 192.168.2.2/24 | Eth0 | 192.168.2.254 | VM2 |
| esx3 | 192.168.3.1/24 | Eth0 | 192.168.3.254 | VM3 |
| esx4 | 192.168.4.1/24 | Eth0 | 192.168.4.254 | VM4 |

### Loopback адреса и NET адреса для BGP (Сеть 10.99.243.0/24)
| Device Name | Loopback Address |
|-------------|------------------|
| 99-blf1 | 10.99.243.1/32 |
| 99-blf2 | 10.99.243.2/32 |
| 99-lf3 | 10.99.243.3/32 |
| 99-lf4 | 10.99.243.4/32 |
| 99-sp1 | 10.99.243.11/32 |
| 99-sp2 | 10.99.243.22/32 |

### Серверные сети 
| Device Name | Server Network | VLAN | Gateway | VM IP |
|-------------|----------------|------|---------|-------|
| 99-blf1 | 192.168.1.0/24 | 10 | 192.168.1.254 | 192.168.1.2 |
| 99-blf2 | 192.168.2.0/24 | 20 | 192.168.2.254 | 192.168.2.2 |
| 99-lf3 | 192.168.3.0/24 | 30 | 192.168.3.254 | 192.168.3.1 |
| 99-lf4 | 192.168.4.0/24 | 40 | 192.168.4.254 | 192.168.4.1 |

---

## Конфигурация IS-IS
## MTU интерфейсов при конфигурации выставлен 1500 , при больших значениях isis не заводится.
### 99-blf1 (Border Leaf 1)
```bash
configure terminal
hostname 99-blf1

ip routing

vlan 10
   name Server-Network-1

interface Ethernet1
   description to-99-sp1-Eth1
   no switchport
   mtu 9100
   ip address 10.99.241.0/31
   bfd interval 300 min-rx 300 multiplier 3

interface Ethernet2
   description to-99-sp2-Eth1
   no switchport
   mtu 9100
   ip address 10.99.242.0/31
   bfd interval 300 min-rx 300 multiplier 3

interface Ethernet3
   description Server-Network1
   switchport access vlan 10
   mtu 9100
   no shutdown

interface Vlan10
   description Server-Network-1
   mtu 9100
   ip address 192.168.1.254/24

interface Loopback0
   description Router-ID
   ip address 10.99.243.1/32

route-map REDISTRIBUTE_CONNECTED permit 10
   match interface Loopback0

route-map REDISTRIBUTE_CONNECTED permit 20
   match interface Vlan10

router bgp 65099
   router-id 10.99.243.1
   no bgp default ipv4-unicast
   timers bgp 3 9
   maximum-paths 2 ecmp 2
   neighbor SPINE peer group
   neighbor SPINE remote-as 65099
   neighbor SPINE timers 3 9
   neighbor SPINE send-community
   neighbor 10.99.241.1 peer group SPINE
   neighbor 10.99.242.1 peer group SPINE
   redistribute connected route-map REDISTRIBUTE_CONNECTED
   
   address-family ipv4
      neighbor SPINE activate

 ```

 ### 99-blf2 (Border Leaf 2)
 ```bash
 configure terminal
hostname 99-blf2

ip routing

vlan 20
   name Server-Network-2

interface Ethernet1
   description to-99-sp1-Eth2
   no switchport
   mtu 9100
   ip address 10.99.241.2/31
   bfd interval 300 min-rx 300 multiplier 3

interface Ethernet2
   description to-99-sp2-Eth2
   no switchport
   mtu 9100
   ip address 10.99.242.2/31
   bfd interval 300 min-rx 300 multiplier 3

interface Ethernet3
   description Server-Network2
   switchport access vlan 20
   mtu 9100
   no shutdown

interface Vlan20
   description Server-Network-2
   mtu 9100
   ip address 192.168.2.254/24

interface Loopback0
   description Router-ID
   ip address 10.99.243.2/32

route-map REDISTRIBUTE_CONNECTED permit 10
   match interface Loopback0

route-map REDISTRIBUTE_CONNECTED permit 20
   match interface Vlan20

router bgp 65099
   router-id 10.99.243.2
   no bgp default ipv4-unicast
   timers bgp 3 9
   maximum-paths 2 ecmp 2
   neighbor SPINE peer group
   neighbor SPINE remote-as 65099
   neighbor SPINE timers 3 9
   neighbor SPINE send-community
   neighbor 10.99.241.3 peer group SPINE
   neighbor 10.99.242.3 peer group SPINE
   redistribute connected route-map REDISTRIBUTE_CONNECTED
   
   address-family ipv4
      neighbor SPINE activate

 ```
### 99-lf3 (Leaf 3)
```bash
configure terminal
hostname 99-lf3

ip routing

vlan 30
   name Server-Network-3

interface Ethernet1
   description to-99-sp1-Eth3
   no switchport
   mtu 9100
   ip address 10.99.241.4/31
   bfd interval 300 min-rx 300 multiplier 3

interface Ethernet2
   description to-99-sp2-Eth3
   no switchport
   mtu 9100
   ip address 10.99.242.4/31
   bfd interval 300 min-rx 300 multiplier 3

interface Ethernet3
   description Server-Network3
   switchport access vlan 30
   mtu 9100
   no shutdown

interface Vlan30
   description Server-Network-3
   mtu 9100
   ip address 192.168.3.254/24

interface Loopback0
   description Router-ID
   ip address 10.99.243.3/32

route-map REDISTRIBUTE_CONNECTED permit 10
   match interface Loopback0

route-map REDISTRIBUTE_CONNECTED permit 20
   match interface Vlan30

router bgp 65099
   router-id 10.99.243.3
   no bgp default ipv4-unicast
   timers bgp 3 9
   maximum-paths 2 ecmp 2
   neighbor SPINE peer group
   neighbor SPINE remote-as 65099
   neighbor SPINE timers 3 9
   neighbor SPINE send-community
   neighbor 10.99.241.5 peer group SPINE
   neighbor 10.99.242.5 peer group SPINE
   redistribute connected route-map REDISTRIBUTE_CONNECTED
   
   address-family ipv4
      neighbor SPINE activate
 ```
 ### 99-lf4 (Leaf 4)
```bash
configure terminal
hostname 99-lf4

ip routing

vlan 40
   name Server-Network-4

interface Ethernet1
   description to-99-sp1-Eth4
   no switchport
   mtu 9100
   ip address 10.99.241.6/31
   bfd interval 300 min-rx 300 multiplier 3

interface Ethernet2
   description to-99-sp2-Eth4
   no switchport
   mtu 9100
   ip address 10.99.242.6/31
   bfd interval 300 min-rx 300 multiplier 3

interface Ethernet3
   description Server-Network4
   switchport access vlan 40
   mtu 9100
   no shutdown

interface Vlan40
   description Server-Network-4
   mtu 9100
   ip address 192.168.4.254/24

interface Loopback0
   description Router-ID
   ip address 10.99.243.4/32

route-map REDISTRIBUTE_CONNECTED permit 10
   match interface Loopback0

route-map REDISTRIBUTE_CONNECTED permit 20
   match interface Vlan40

router bgp 65099
   router-id 10.99.243.4
   no bgp default ipv4-unicast
   timers bgp 3 9
   maximum-paths 2 ecmp 2
   neighbor SPINE peer group
   neighbor SPINE remote-as 65099
   neighbor SPINE timers 3 9
   neighbor SPINE send-community
   neighbor 10.99.241.7 peer group SPINE
   neighbor 10.99.242.7 peer group SPINE
   redistribute connected route-map REDISTRIBUTE_CONNECTED
   
   address-family ipv4
      neighbor SPINE activate
 ```

 ### 99-sp1 (Spine 1)
 ```bash
configure terminal
hostname 99-sp1

! Настройка IP Routing
ip routing

! Настройка интерфейсов
interface Ethernet1
   description to-99-blf1-Eth1
   no switchport
   mtu 9100
   ip address 10.99.241.1/31
   bfd interval 300 min-rx 300 multiplier 3

interface Ethernet2
   description to-99-blf2-Eth1
   no switchport
   mtu 9100
   ip address 10.99.241.3/31
   bfd interval 300 min-rx 300 multiplier 3

interface Ethernet3
   description to-99-lf3-Eth1
   no switchport
   mtu 9100
   ip address 10.99.241.5/31
   bfd interval 300 min-rx 300 multiplier 3

interface Ethernet4
   description to-99-lf4-Eth1
   no switchport
   mtu 9100
   ip address 10.99.241.7/31
   bfd interval 300 min-rx 300 multiplier 3

interface Loopback0
   description Router-ID
   ip address 10.99.243.11/32

! Настройка BGP (Route Reflector Server)
router bgp 65099
   router-id 10.99.243.11
   no bgp default ipv4-unicast
   timers bgp 3 9
   maximum-paths 2 ecmp 2
   neighbor LEAF peer group
   neighbor LEAF remote-as 65099
   neighbor LEAF route-reflector-client
   neighbor LEAF send-community
   neighbor 10.99.241.0 peer group LEAF
   neighbor 10.99.241.2 peer group LEAF
   neighbor 10.99.241.4 peer group LEAF
   neighbor 10.99.241.6 peer group LEAF
   
   address-family ipv4
      neighbor LEAF activate
      network 10.99.243.11/32
 ```

 ### 99-sp2 (Spine 2)
 ```bash
configure terminal
hostname 99-sp2

! Настройка IP Routing
ip routing

! Настройка интерфейсов
interface Ethernet1
   description to-99-blf1-Eth2
   no switchport
   mtu 9100
   ip address 10.99.242.1/31
   bfd interval 300 min-rx 300 multiplier 3

interface Ethernet2
   description to-99-blf2-Eth2
   no switchport
   mtu 9100
   ip address 10.99.242.3/31
   bfd interval 300 min-rx 300 multiplier 3

interface Ethernet3
   description to-99-lf3-Eth2
   no switchport
   mtu 9100
   ip address 10.99.242.5/31
   bfd interval 300 min-rx 300 multiplier 3

interface Ethernet4
   description to-99-lf4-Eth2
   no switchport
   mtu 9100
   ip address 10.99.242.7/31
   bfd interval 300 min-rx 300 multiplier 3

interface Loopback0
   description Router-ID
   ip address 10.99.243.22/32

! Настройка BGP (Route Reflector Server)
router bgp 65099
   router-id 10.99.243.22
   no bgp default ipv4-unicast
   timers bgp 3 9
   maximum-paths 2 ecmp 2
   neighbor LEAF peer group
   neighbor LEAF remote-as 65099
   neighbor LEAF route-reflector-client
   neighbor LEAF send-community
   neighbor 10.99.242.0 peer group LEAF
   neighbor 10.99.242.2 peer group LEAF
   neighbor 10.99.242.4 peer group LEAF
   neighbor 10.99.242.6 peer group LEAF
   
   address-family ipv4
      neighbor LEAF activate
      network 10.99.243.22/32

 ```
---

## Проверка IP связности
 ## 1. Проверка IS-IS соседств
```
99-sp1#sh isis neighbors

Instance  VRF      System Id        Type Interface          SNPA              State Hold time   Circuit Id
UNDERLAY  default  99-blf1          L2   Ethernet1          P2P               UP    28          1A     
UNDERLAY  default  99-blf2          L2   Ethernet2          P2P               UP    28          14     
UNDERLAY  default  99-lf3           L2   Ethernet3          P2P               UP    22          1A     

```
## 2.  Проверка IS-IS database 
```
99-sp1#sh isis database
Legend:
H - hostname conflict
U - node unreachable

IS-IS Instance: UNDERLAY VRF: default
  IS-IS Level 2 Link State Database
    LSPID                   Seq Num  Cksum  Life Length IS  Received LSPID        Flags
    99-blf1.00-00                19  49448   976    133 L2  0100.9924.3001.00-00  <>
    99-blf2.00-00                10   3287   727    133 L2  0100.9924.3002.00-00  <>
    99-lf3.00-00                  4   9243   591    132 L2  0100.9924.3003.00-00  <>
    99-sp1.00-00                 15   8500   976    146 L2  0100.9924.3011.00-00  <>
    99-sp2.00-00                  7   2840   907    146 L2  0100.9924.3022.00-00  <>
```

## 2. Проверка IS-IS маршрутов
```
99-sp1#sh ip route isis

VRF: default

 I L2     10.99.242.0/31 [115/20]
           via 10.99.241.0, Ethernet1
 I L2     10.99.242.2/31 [115/20]
           via 10.99.241.2, Ethernet2
 I L2     10.99.242.4/31 [115/20]
           via 10.99.241.4, Ethernet3
 I L2     10.99.243.1/32 [115/20]
           via 10.99.241.0, Ethernet1
 I L2     10.99.243.2/32 [115/20]
           via 10.99.241.2, Ethernet2
 I L2     10.99.243.3/32 [115/20]
           via 10.99.241.4, Ethernet3
 I L2     10.99.243.22/32 [115/30]
           via 10.99.241.0, Ethernet1
           via 10.99.241.2, Ethernet2
           via 10.99.241.4, Ethernet3
 I L2     192.168.1.0/24 [115/20]
           via 10.99.241.0, Ethernet1
 I L2     192.168.2.0/24 [115/20]
           via 10.99.241.2, Ethernet2
 I L2     192.168.3.0/24 [115/20]
           via 10.99.241.4, Ethernet3

```
## 3. Проверка межсерверной связности между VM 
 ![Проверка пингом между ВМ](linux_ping.jpg)

## 4. Проверка связности между loopback адресами
```
99-lf3#ping 10.99.243.1 source 10.99.243.3
PING 10.99.243.1 (10.99.243.1) from 10.99.243.3 : 72(100) bytes of data.
80 bytes from 10.99.243.1: icmp_seq=1 ttl=63 time=3.16 ms
80 bytes from 10.99.243.1: icmp_seq=2 ttl=63 time=1.82 ms
80 bytes from 10.99.243.1: icmp_seq=3 ttl=63 time=1.81 ms
80 bytes from 10.99.243.1: icmp_seq=4 ttl=63 time=1.80 ms
80 bytes from 10.99.243.1: icmp_seq=5 ttl=63 time=1.82 ms

--- 10.99.243.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 13ms
rtt min/avg/max/mdev = 1.795/2.081/3.162/0.540 ms, ipg/ewma 3.314/2.603 ms

99-lf3#
99-lf3#ping 10.99.243.2 source 10.99.243.3
PING 10.99.243.2 (10.99.243.2) from 10.99.243.3 : 72(100) bytes of data.
80 bytes from 10.99.243.2: icmp_seq=1 ttl=63 time=4.24 ms
80 bytes from 10.99.243.2: icmp_seq=2 ttl=63 time=2.71 ms
80 bytes from 10.99.243.2: icmp_seq=3 ttl=63 time=2.47 ms
80 bytes from 10.99.243.2: icmp_seq=4 ttl=63 time=2.36 ms
80 bytes from 10.99.243.2: icmp_seq=5 ttl=63 time=2.27 ms

--- 10.99.243.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 17ms
rtt min/avg/max/mdev = 2.271/2.809/4.240/0.730 ms, ipg/ewma 4.361/3.490 ms

99-lf3#ping 10.99.243.11 source 10.99.243.3
PING 10.99.243.11 (10.99.243.11) from 10.99.243.3 : 72(100) bytes of data.
80 bytes from 10.99.243.11: icmp_seq=1 ttl=64 time=1.90 ms
80 bytes from 10.99.243.11: icmp_seq=2 ttl=64 time=0.871 ms
80 bytes from 10.99.243.11: icmp_seq=3 ttl=64 time=0.838 ms
80 bytes from 10.99.243.11: icmp_seq=4 ttl=64 time=0.768 ms
80 bytes from 10.99.243.11: icmp_seq=5 ttl=64 time=0.881 ms

--- 10.99.243.11 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 10ms
rtt min/avg/max/mdev = 0.768/1.050/1.895/0.424 ms, ipg/ewma 2.490/1.458 ms
99-lf3#
99-lf3#ping 10.99.243.22 source 10.99.243.3
PING 10.99.243.22 (10.99.243.22) from 10.99.243.3 : 72(100) bytes of data.
80 bytes from 10.99.243.22: icmp_seq=1 ttl=64 time=3.17 ms
80 bytes from 10.99.243.22: icmp_seq=2 ttl=64 time=1.94 ms
80 bytes from 10.99.243.22: icmp_seq=3 ttl=64 time=1.96 ms
80 bytes from 10.99.243.22: icmp_seq=4 ttl=64 time=1.85 ms
80 bytes from 10.99.243.22: icmp_seq=5 ttl=64 time=0.833 ms

--- 10.99.243.22 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 14ms
rtt min/avg/max/mdev = 0.833/1.949/3.166/0.739 ms, ipg/ewma 3.409/2.512 ms
```
