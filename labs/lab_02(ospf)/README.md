# Лабораторная работа: Настройка Underlay сети с использованием OSPF

## Задание
1. Настроить OSPF в Underlay сети для обеспечения IP-связности между всеми сетевыми устройствами
2. Задокументировать:
   - План работы
   - Схему сети
   - Конфигурацию устройств
3. Проверить IP-связность между устройствами в OSPF домене

---

## Топология сети

## IP-план (Address Plan)

### Underlay сеть (Fabric Links - Point-to-Point /31)
| Device Name | IP Address/Маска | Port | Remote Device | Remote Port | Description |
|-------------|------------------|------|---------------|-------------|-------------|
| 99-blf1 | 10.99.241.0/31 | Ethernet1 | 99-sp1 | Ethernet1 | to Spine1 |
| 99-blf1 | 10.99.242.0/31 | Ethernet2 | 99-sp2 | Ethernet1 | to Spine2 |
| 99-blf1 | 192.168.1.254/24 | Ethernet3 | Linux1 | Eth0 | Server Network1 |
| 99-blf2 | 10.99.241.2/31 | Ethernet1 | 99-sp1 | Ethernet2 | to Spine1 |
| 99-blf2 | 10.99.242.2/31 | Ethernet2 | 99-sp2 | Ethernet2 | to Spine2 |
| 99-blf2 | 192.168.2.254/24 | Ethernet3 | Linux2 | Eth0 | Server Network2 |
| 99-lf3 | 10.99.241.4/31 | Ethernet1 | 99-sp1 | Ethernet3 | to Spine1 |
| 99-lf3 | 10.99.242.4/31 | Ethernet2 | 99-sp2 | Ethernet3 | to Spine2 |
| 99-lf3 | 192.168.3.254/24 | Ethernet3 | Linux3 | Eth0 | Server Network3 |
| 99-lf3 | 192.168.3.254/24 | Ethernet4 | Linux4 | Eth0 | Server Network3 |
| 99-sp1 | 10.99.241.1/31 | Ethernet1 | 99-blf1 | Ethernet1 | to BorderLeaf1 |
| 99-sp1 | 10.99.241.3/31 | Ethernet2 | 99-blf2 | Ethernet1 | to BorderLeaf2 |
| 99-sp1 | 10.99.241.5/31 | Ethernet3 | 99-lf3 | Ethernet1 | to Leaf3 |
| 99-sp2 | 10.99.242.1/31 | Ethernet1 | 99-blf1 | Ethernet2 | to BorderLeaf1 |
| 99-sp2 | 10.99.242.3/31 | Ethernet2 | 99-blf2 | Ethernet2 | to BorderLeaf2 |
| 99-sp2 | 10.99.242.5/31 | Ethernet3 | 99-lf3 | Ethernet2 | to Leaf3 |

### Серверные ВМ
| Device Name | IP Address/Маска | Port | Gateway | Description |
|-------------|------------------|------|---------|-------------|
| Linux1 | 192.168.1.2/24 | Eth0 | 192.168.1.254 | VM1 |
| Linux2 | 192.168.2.2/24 | Eth0 | 192.168.2.254 | VM2 |
| Linux3 | 192.168.1.1/24 | Eth0 | 192.168.1.254 | VM3 |
| Linux4 | 192.168.2.1/24 | Eth0 | 192.168.2.254 | VM4 |

### Loopback адреса для OSPF (Сеть 10.99.243.0/24) - Area 0
| Device Name | Loopback Address | Router-ID | Description |
|-------------|------------------|-----------|-------------|
| 99-blf1 | 10.99.243.1/32 | 10.99.243.1 | Border Leaf 1 Loopback |
| 99-blf2 | 10.99.243.2/32 | 10.99.243.2 | Border Leaf 2 Loopback |
| 99-lf3 | 10.99.243.3/32 | 10.99.243.3 | Leaf 3 Loopback |
| 99-sp1 | 10.99.243.11/32 | 10.99.243.11 | Spine 1 Loopback |
| 99-sp2 | 10.99.243.22/32 | 10.99.243.22 | Spine 2 Loopback |

### Серверные сети - Area 10
| Device Name | Server Network | VLAN | Gateway | VM IP | Area |
|-------------|----------------|------|---------|-------|------|
| 99-blf1 | 192.168.1.0/24 | 10 | 192.168.1.254 | 192.168.1.2 | Area 10 |
| 99-blf2 | 192.168.2.0/24 | 20 | 192.168.2.254 | 192.168.2.2 | Area 10 |
| 99-lf3 | 192.168.3.0/24 | 30 | 192.168.3.254 | 192.168.3.1 | Area 10 |
| 99-lf3 | 192.168.2.0/24 | 30 | 192.168.3.254 | 192.168.3.2 | Area 10 |
---

## Конфигурация OSPF

### 99-blf1 (Border Leaf 1)
```bash
configure terminal
interface Loopback0
 description OSPF Router-ID and Underlay Management
 ip address 10.99.243.1/32
 no shutdown

interface Vlan10
 description Server-Network-1
 ip address 192.168.1.254/24
 no shutdown

router ospf UNDERLAY
 router-id 10.99.243.1
 maximum-paths 4
 network 10.99.243.1/32 area 0.0.0.0
 network 192.168.1.0/24 area 0.0.0.10
 passive-interface Vlan10

interface Ethernet1
 ip ospf network point-to-point
 ip ospf hello-interval 1
 ip ospf dead-interval 3
 
interface Ethernet2
 ip ospf network point-to-point
 ip ospf hello-interval 1
 ip ospf dead-interval 3
 ```