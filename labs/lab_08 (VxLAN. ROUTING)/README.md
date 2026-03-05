# Лабораторная работа:  Реализовать передачу суммарных префиксов через EVPN route-type 5

## Задание
1. Разместить каждый Vlan ( 10,20,30,40) в своем VRF
2. Затерминировать VRF на Фаерволле
3. Собрать отказоустойчивый кластер USG 6000
4. Настроить маршрутизацию между VRF через фаерволл

---

## Топология сети
![Схема Сети ](fabric_scheme.jpg)

Добавим в IP-план линки до фаерволлов, и адреса для пиринга с ними

## IP-план (Address Plan) 

### Underlay сеть (Fabric Links - Point-to-Point /31)

| Device Name | IP Address/Маска | Port      | Remote Device | Remote Port | Description         |
|-------------|------------------|-----------|---------------|-------------|---------------------|
| 99-blf1     | 10.99.241.0/31   | Ethernet1 | 99-sp1        | Ethernet1   | to Spine1           |
| 99-blf1     | 10.99.242.0/31   | Ethernet2 | 99-sp2        | Ethernet1   | to Spine2           |
| 99-blf1     | -                | Ethernet3 | 99-esx1       | Ethernet1   | Server Trunk        |
| 99-blf1     | -                | Ethernet4 | 99-fw01       | GI1/0/0     | to fw01             |
| 99-blf1     | -                | Ethernet5 | 99-esx2       | Ethernet2   | Server Trunk        |
| 99-blf2     | 10.99.241.2/31   | Ethernet1 | 99-sp1        | Ethernet2   | to Spine1           |
| 99-blf2     | 10.99.242.2/31   | Ethernet2 | 99-sp2        | Ethernet2   | to Spine2           |
| 99-blf2     | -                | Ethernet3 | 99-esx1       | Ethernet1   | Server Trunk        |
| 99-blf2     | -                | Ethernet5 | 99-esx2       | Ethernet2   | Server Trunk        |
| 99-blf2     | -                | Ethernet4 | 99-fw02       | GI1/0/1     | to fw02             |
| 99-lf3      | 10.99.241.4/31   | Ethernet1 | 99-sp1        | Ethernet3   | to Spine1           |
| 99-lf3      | 10.99.242.4/31   | Ethernet2 | 99-sp2        | Ethernet3   | to Spine2           |
| 99-lf3      | -                | Ethernet3 | 99-esx3       | Ethernet1   | Server Trunk        |
| 99-lf3      | -                | Ethernet4 | 99-esx4       | Ethernet2   | Server Trunk        |
| 99-lf4      | 10.99.241.6/31   | Ethernet1 | 99-sp1        | Ethernet4   | to Spine1           |
| 99-lf4      | 10.99.242.6/31   | Ethernet2 | 99-sp2        | Ethernet4   | to Spine2           |
| 99-lf4      | -                | Ethernet3 | 99-esx3       | Ethernet2   | Server Trunk        |
| 99-lf4      | -                | Ethernet4 | 99-esx4       | Ethernet1   | Server Trunk        |
| 99-fw01     | 10.99.242.12/31  |Eth-Trunk23| 99-fw02       |Eth-Trunk23  | HRP LINK            |

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

### P2P адреса для VRF (Сеть 10.99.1.0/24)

| Device Name | Loopback0 Address | VRF         |
|-------------|-------------------|-------------|
| 99-fw01     | 10.99.1.0/31      | VRF_CORE_1  |
| 99-fw02     | 10.99.1.3/31      | VRF_CORE_1  |
| 99-blf1     | 10.99.1.1/31      | VRF_CORE_1  |
| 99-blf2     | 10.99.1.2/31      | VRF_CORE_1  |
| 99-fw01     | 10.99.2.0/31      | VRF_CORE_2  |
| 99-fw02     | 10.99.2.3/31      | VRF_CORE_2  |
| 99-blf1     | 10.99.2.1/31      | VRF_CORE_2  |
| 99-blf2     | 10.99.2.2/31      | VRF_CORE_2  |
| 99-fw01     | 10.99.3.0/31      | VRF_CORE_3  |
| 99-fw02     | 10.99.3.3/31      | VRF_CORE_3  |
| 99-blf1     | 10.99.3.1/31      | VRF_CORE_3  |
| 99-blf2     | 10.99.3.2/31      | VRF_CORE_3  |
| 99-fw01     | 10.99.4.0/31      | VRF_CORE_4  |
| 99-fw02     | 10.99.4.3/31      | VRF_CORE_4  |
| 99-blf1     | 10.99.4.1/31      | VRF_CORE_4  |
| 99-blf2     | 10.99.4.2/31      | VRF_CORE_4  |

### MGMT адреса (Сеть 10.99.245.0/24)

| Device Name | Loopback0 Address | Description   |
|-------------|-------------------|---------------|
| 99-blf1     | 10.99.245.1/32    | Mgmt ip       |
| 99-blf2     | 10.99.245.2/32    | Mgmt ip       |
| 99-lf3      | 10.99.245.3/32    | Mgmt ip       |
| 99-lf4      | 10.99.245.4/32    | Mgmt ip       |
| 99-sp1      | 10.99.245.11/32   | Mgmt ip       |
| 99-sp2      | 10.99.245.22/32   | Mgmt ip       |
| 99-fw01     | 10.99.245.252/32  | Mgmt ip       |
| 99-fw02     | 10.99.245.253/32  | Mgmt ip       |
| 99-fw       | 10.99.245.254/32  | VIP GW        |


### EVPN L2-домены (VLAN ↔ VNI Mapping)

| VLAN | Описание          | L2 VNI | Anycast Gateway   | Leaf с настроенным VNI |
|------|-------------------|--------|-------------------|------------------------|
| 10   | VLAN10   | 10010  | 192.168.1.254/24  | 99-blf1, 99-blf2 VRF_CORE1 |
| 20   | VLAN20   | 10020  | 192.168.2.254/24  | 99-blf1, 99-blf2 VRF_CORE2 |
| 30   | VLAN30   | 10030  | 192.168.3.254/24  | 99-lf3, 99-lf4 VRF_CORE3 |
| 40   | VLAN40   | 10040  | 192.168.4.254/24  | 99-lf3, 99-lf4 VRF_CORE4 |



### Конфигурация "ESXi" устройств 

| ESXi Device | Физические порты (Trunk) | Виртуальные интерфейсы (SVI)                     | IP Address/Маска   | Gateway         |
|-------------|--------------------------|------------------------------------------------|-------------------|-----------------|
| 99-esx1     | Eth1 → 99-blf1 Eth3   Po1   | Vlan10 (SVI для VLAN10)                        | 192.168.1.1/24    | 192.168.1.254  |
| 99-esx1     | Eth2 → 99-blf2 Eth3   Po1   | Vlan10 (SVI для VLAN10)                        | 192.168.1.1/24    | 192.168.1.254  |
| 99-esx2     | Eth1 → 99-blf1 Eth5   Po2  | Vlan10 (SVI для VLAN10)                        | 192.168.1.2/24    | 192.168.1.254   |
| 99-esx2     | Eth1 → 99-blf2 Eth5    Po2   | Vlan20 (SVI для VLAN20)                        | 192.168.2.1/24    | 192.168.2.254   |
| 99-esx2     | Eth1 → 99-blf1 Eth5   Po2  | Vlan10 (SVI для VLAN10)                        | 192.168.1.2/24    | 192.168.1.254   |
| 99-esx2     | Eth1 → 99-blf2 Eth5    Po2   | Vlan20 (SVI для VLAN20)                        | 192.168.2.1/24    | 192.168.2.254   |
| 99-esx3     | Eth1 → 99-lf3 Eth3   Po1   | Vlan30 (SVI для VLAN30)                        | 192.168.3.1/24    | 192.168.3.254   |
| 99-esx3     | Eth1 → 99-lf4 Eth3   Po1    | Vlan30 (SVI для VLAN30)                        | 192.168.3.1/24    | 192.168.3.254   |
| 99-esx4     | Eth1 → 99-lf3 Eth5    Po1   | Vlan40 (SVI для VLAN40)                        | 192.168.4.1/24    | 192.168.4.254   |
| 99-esx4     | Eth1 → 99-lf4 Eth5    Po1   | Vlan40 (SVI для VLAN40)                        | 192.168.4.1/24    | 192.168.4.254  |

---

## Конфигурация 

## FW01/02
Для начала настроим доступ по ssh с blf поскольку нормальный доступ на хувиках в лабе не заработал, потому что после включения нужно прожать кнопку энтер чтоб устройство загрузилось, а в ssh режиме этого сделать не выходит.

<details>
<summary>Первоначальная настройка FW </summary>

```bash
system-config
sysname 99-fw01

interface GigabitEthernet0/0/0
 undo shutdown
 ip binding vpn-instance default
 ip address 10.99.245.252 255.255.255.0
 service-manage http permit
 service-manage https permit
 service-manage ping permit
 service-manage ssh permit

stelnet server enable
ssh authentication-type default password
user-interface vty 0 4
 authentication-mode aaa
 protocol inbound ssh
aaa
   manager-user admin
  password cipher *******
  service-type ssh
  q
 firewall zone trust
 set priority 85
 add interface GigabitEthernet0/0/0
 q
q
save 
```
```bash
system-config
sysname 99-fw02

interface GigabitEthernet0/0/0
 undo shutdown
 ip binding vpn-instance default
 ip address 10.99.245.253 255.255.255.0
 service-manage http permit
 service-manage https permit
 service-manage ping permit
 service-manage ssh permit
 
stelnet server enable
ssh authentication-type default password
user-interface vty 0 4
 authentication-mode aaa
 protocol inbound ssh
aaa
   manager-user admin
  password cipher *******
  service-type ssh
  q
 firewall zone trust
 set priority 85
 add interface GigabitEthernet0/0/0
q
q
save
```
</details>

Все, теперь мы можем зайти на них с любого из наших свичей по ssh или по http на менеджмент адрес 

## Поднимем HRP
при настройке HRP зададим руками значения MED для эктив стендбай ноды, чтоб в дальнейшем на фабрике выбирался дефолт с активной ноды

<details>
<summary>Настройка HRP </summary>

99-fw01
```bash
interface Eth-Trunk23
ip address 10.99.241.12 255.255.255.254
description HRP LINK to 99-fw02
service-manage all permit
q
interface Gi1/0/2
eth-trunk 23
description HRP LINK to 99-fw02
interface Gi1/0/3
eth-trunk 23
description HRP LINK to 99-fw02
q
firewall zone trust 
add int Eth-Trunk23

hrp enable
hrp standby config enable
hrp interface Eth-Trunk23 remote 10.99.241.13
hrp track interface Eth-Trunk 23
hrp adjust bgp-cost enable 10

```
99-fw02
```bash
interface Eth-Trunk23
ip address 10.99.241.13 255.255.255.254
description HRP LINK to 99-fw01
service-manage all permit
q
interface Gi1/0/2
eth-trunk 23
description HRP LINK to 99-fw01
interface Gi1/0/3
eth-trunk 23
description HRP LINK to 99-fw01
q
firewall zone trust 
add int Eth-Trunk23

hrp enable
hrp standby config enable
hrp standby-device
hrp interface Eth-Trunk23 remote 10.99.241.12
hrp track interface Eth-Trunk 23
hrp adjust bgp-cost enable 20
```
</details>

Проверим:
```bash
HRP_S<99-fw02>disp hrp state
 Role: standby, peer: active
 Running priority: 45000, peer: 45000
 Core state: abnormal(standby), peer: abnormal(active)
 Backup channel usage: 0.00%
 Stable time: 0 days, 0 hours, 0 minutes
 Last state change information: 2026-03-01 23:46:52 HRP link changes to up.

```


### Настройка VRF на фабрике 
--------------------------------------------------------------------------------------------------------
Подправим конфиг на лифах - вынесем каждую подсеть 192.168.N.0/24 в свой VRF и добавим 3/4 VRF на 1/2 blf для настройки связности их с фаерволлом

<details>
<summary>Конфиг </summary>

blf01/02
```bash
vrf instance VRF_CORE_3
vrf instance VRF_CORE_4

ip routing vrf MGMT
ip routing vrf VRF_CORE_1
ip routing vrf VRF_CORE_2
ip routing vrf VRF_CORE_3
ip routing vrf VRF_CORE_4

int vxlan1
   vxlan vrf VRF_CORE_1 vni 10001
   vxlan vrf VRF_CORE_2 vni 10002
   vxlan vrf VRF_CORE_3 vni 10003
   vxlan vrf VRF_CORE_4 vni 10004


router bgp 65099
vrf VRF_CORE_3
      rd 65099:1031  ( rd 65099:1032 для второго лифа)
      route-target import evpn 65099:103
      route-target export evpn 65099:103
           address-family ipv4
         redistribute connected
ex
ex
vrf VRF_CORE_4
      rd 65099:1041 ( rd 65099:1042 для второго лифа)
      route-target import evpn 65099:104
      route-target export evpn 65099:104
      address-family ipv4
         redistribute connected
ex
ex
ip route vrf MGMT 0.0.0.0/0 10.99.245.254

interface Vlan10
   description Vlan10
   mtu 9100
   vrf VRF_CORE_1
   ip address virtual 192.168.1.254/24
ex
interface Vlan20
   description Vlan20
   mtu 9100
   vrf VRF_CORE_2
   ip address virtual 192.168.2.254/24

```
lf03/04
```bash
vrf instance VRF_CORE_3
vrf instance VRF_CORE_4

ip routing vrf MGMT
ip routing vrf VRF_CORE_3
ip routing vrf VRF_CORE_4


router bgp 65099
vrf VRF_CORE_3
      rd 65099:1033 /1034
      route-target import evpn 65099:103
      route-target export evpn 65099:103
           address-family ipv4
         redistribute connected
ex
ex
vrf VRF_CORE_4
      rd 65099:1044 /1044
      route-target import evpn 65099:104 
      route-target export evpn 65099:104
      address-family ipv4
         redistribute connected
ex
ex
ip route vrf MGMT 0.0.0.0/0 10.99.245.254

interface Vlan30
   description Vlan30
   mtu 9100
   vrf VRF_CORE_3
   ip address virtual 192.168.3.254/24
ex
interface Vlan40
   description Vlan40
   mtu 9100
   vrf VRF_CORE_4
   ip address virtual 192.168.4.254/24

```
</details>

### Настройка линков и маршрутизации на blf01/02
В идеале нужно коммутировать каждый фаерволл двумя линками к каждому blf, но на аристах осталось всего 2 порта один из которых запланирован под мультисайт, потому пришлось скоммутировать одним линком.
Но для того чтоб не было путанницы с двумя разными дефолтами на каждом blf настроим по два пиринга с каждого фаерволла до блифов.
Для того чтоб блифы могли заанонсить нормально друг другу p2p сети с фаерволлом внутри VRF поменяем айпи vtepа на втором лифе на 10.99.244.2 - в противном случае маршруты не инсталлятся в таблицу. Либо можно оставить общий Vtep но придется тогда строить пиринг внутри VRF между блифами по пирлинку, что выглядит хуже.

<details>
<summary>Настройка линков и маршрутизации на blf01/02 </summary>

общее
```bash
ip prefix-list DEFAULT
 seq 10 permit 0.0.0.0/0
route-map DEFAULT
 match ip address prefix-list DEFAULT
 ```

blf01
```bash
int et 4
no switchport
description 99-fw01 G1/0/0
no shut
interface Ethernet4.10
   encapsulation dot1q vlan 10
   vrf VRF_CORE_1
   ip address 10.99.1.1/31
!
interface Ethernet4.20
   encapsulation dot1q vlan 20
   vrf VRF_CORE_2
   ip address 10.99.2.1/31
!
interface Ethernet4.30
   encapsulation dot1q vlan 30
   vrf VRF_CORE_3
   ip address 10.99.3.1/31
!
interface Ethernet4.40
   encapsulation dot1q vlan 40
   vrf VRF_CORE_4
   ip address 10.99.4.1/31

router bgp 65099
 vrf VRF_CORE_1
      rd 65099:1011
      route-target import evpn 65099:101
      route-target export evpn 65099:101
      router-id 10.99.1.1
      neighbor 10.99.1.0 remote-as 65098
      neighbor 10.99.1.0 next-hop-self
      neighbor 10.99.1.0 update-source Ethernet4.10
      neighbor 10.99.1.0 route-map DEFAULT in
      neighbor 10.99.1.3 remote-as 65098
      neighbor 10.99.1.3 next-hop-self
      neighbor 10.99.1.3 update-source Ethernet4.10
      neighbor 10.99.1.3 ebgp-multihop 10
      neighbor 10.99.1.3 route-map DEFAULT in
      !
      address-family ipv4
         neighbor 10.99.1.0 activate
         neighbor 10.99.1.3 activate
         network 10.99.1.0/31
         redistribute connected


vrf VRF_CORE_2
      rd 65099:1021
      route-target import evpn 65099:102
      route-target export evpn 65099:102
      router-id 10.99.2.1
      neighbor 10.99.2.0 remote-as 65098
      neighbor 10.99.2.0 next-hop-self
      neighbor 10.99.2.0 update-source Ethernet4.20
      neighbor 10.99.2.0 route-map DEFAULT in
      neighbor 10.99.2.3 remote-as 65098
      neighbor 10.99.2.3 next-hop-self
      neighbor 10.99.2.3 update-source Ethernet4.20
      neighbor 10.99.2.3 ebgp-multihop 10
      neighbor 10.99.2.3 route-map DEFAULT in
      !
      address-family ipv4
         neighbor 10.99.2.0 activate
         neighbor 10.99.2.3 activate
         network 10.99.2.0/31
         redistribute connected

vrf VRF_CORE_3
      rd 65099:1031
      route-target import evpn 65099:103
      route-target export evpn 65099:103
      router-id 10.99.3.1
      neighbor 10.99.3.0 remote-as 65098
      neighbor 10.99.3.0 next-hop-self
      neighbor 10.99.3.0 update-source Ethernet4.30
      neighbor 10.99.3.0 route-map DEFAULT in
      neighbor 10.99.3.3 remote-as 65098
      neighbor 10.99.3.3 next-hop-self
      neighbor 10.99.3.3 update-source Ethernet4.30
      neighbor 10.99.3.3 ebgp-multihop 10
      neighbor 10.99.3.3 route-map DEFAULT in
      !
      address-family ipv4
         neighbor 10.99.3.0 activate
         neighbor 10.99.3.3 activate
         network 10.99.3.0/31
         redistribute connected

vrf VRF_CORE_4
      rd 65099:1041
      route-target import evpn 65099:104
      route-target export evpn 65099:104
      router-id 10.99.4.1
      neighbor 10.99.4.0 remote-as 65098
      neighbor 10.99.4.0 next-hop-self
      neighbor 10.99.4.0 update-source Ethernet4.40
      neighbor 10.99.4.0 route-map DEFAULT in
      neighbor 10.99.4.3 remote-as 65098
      neighbor 10.99.4.3 next-hop-self
      neighbor 10.99.4.3 update-source Ethernet4.40
      neighbor 10.99.4.3 ebgp-multihop 10
      neighbor 10.99.4.3 route-map DEFAULT in
      !
      address-family ipv4
         neighbor 10.99.4.0 activate
         neighbor 10.99.4.3 activate
         network 10.99.4.0/31
         redistribute connected
    

```
blf02
```bash
int et 4
no switchport
description 99-fw01 G1/0/0
no shut
int et4.10
encapsulation dot1q vlan 10
vrf VRF_CORE_1
ip address 10.99.1.2/31
int et4.20
encapsulation dot1q vlan 20
vrf VRF_CORE_2
ip address 10.99.2.2/31
int et4.30
encapsulation dot1q vlan 30
vrf VRF_CORE_3
ip address 10.99.3.2/31
int et4.40
encapsulation dot1q vlan 40
vrf VRF_CORE_4
ip address 10.99.4.2/31

router bgp 65099
 vrf VRF_CORE_1
      rd 65099:1012
      route-target import evpn 65099:101
      route-target export evpn 65099:101
      router-id 10.99.1.2
      neighbor 10.99.1.0 remote-as 65098
      neighbor 10.99.1.0 next-hop-self
      neighbor 10.99.1.0 update-source Ethernet4.10
      neighbor 10.99.1.0 ebgp-multihop 10
      neighbor 10.99.1.0 route-map DEFAULT in
      neighbor 10.99.1.3 remote-as 65098
      neighbor 10.99.1.3 next-hop-self
      neighbor 10.99.1.3 update-source Ethernet4.10
      neighbor 10.99.1.3 route-map DEFAULT in
      !
      address-family ipv4
         neighbor 10.99.1.0 activate
         neighbor 10.99.1.3 activate
         network 10.99.1.2/31
         redistribute connected

vrf VRF_CORE_2
      rd 65099:1022
      route-target import evpn 65099:102
      route-target export evpn 65099:102
      router-id 10.99.2.2
      neighbor 10.99.2.0 remote-as 65098
      neighbor 10.99.2.0 next-hop-self
      neighbor 10.99.2.0 update-source Ethernet4.20
      neighbor 10.99.2.0 ebgp-multihop 10
      neighbor 10.99.2.0 route-map DEFAULT in
      neighbor 10.99.2.3 remote-as 65098
      neighbor 10.99.2.3 next-hop-self
      neighbor 10.99.2.3 update-source Ethernet4.20
      neighbor 10.99.2.3 route-map DEFAULT in
      !
      address-family ipv4
         neighbor 10.99.2.0 activate
         neighbor 10.99.2.3 activate
         network 10.99.2.2/31
         redistribute connected

vrf VRF_CORE_3
      rd 65099:1032
      route-target import evpn 65099:103
      route-target export evpn 65099:103
      router-id 10.99.3.2
      neighbor 10.99.3.0 remote-as 65098
      neighbor 10.99.3.0 next-hop-self
      neighbor 10.99.3.0 update-source Ethernet4.30
      neighbor 10.99.3.0 ebgp-multihop 10
      neighbor 10.99.3.0 route-map DEFAULT in
      neighbor 10.99.3.3 remote-as 65098
      neighbor 10.99.3.3 next-hop-self
      neighbor 10.99.3.3 update-source Ethernet4.30
      neighbor 10.99.3.3 route-map DEFAULT in
      !
      address-family ipv4
         neighbor 10.99.3.0 activate
         neighbor 10.99.3.3 activate
         network 10.99.3.2/31
         redistribute connected

vrf VRF_CORE_4
      rd 65099:1042
      route-target import evpn 65099:104
      route-target export evpn 65099:104
      router-id 10.99.4.2
      neighbor 10.99.4.0 remote-as 65098
      neighbor 10.99.4.0 next-hop-self
      neighbor 10.99.4.0 update-source Ethernet4.40
      neighbor 10.99.4.0 ebgp-multihop 10
      neighbor 10.99.4.0 route-map DEFAULT in
      neighbor 10.99.4.3 remote-as 65098
      neighbor 10.99.4.3 next-hop-self
      neighbor 10.99.4.3 update-source Ethernet4.40
      neighbor 10.99.4.3 route-map DEFAULT in
      !
      address-family ipv4
         neighbor 10.99.4.0 activate
         neighbor 10.99.4.3 activate
         network 10.99.4.2/31
         redistribute connected
```
</details>

### Настройка линков и маршрутизации на фаерволле

Настроим пиринги, и сделаем дефолтный статик для анонсов на фабрику

<details>
<summary>Конфиг</summary>

fw01
```bash

interface GigabitEthernet1/0/0
 description 99blf01-Et4
 undo shutdown
 service-manage ping permit
#
interface GigabitEthernet1/0/0.10
 vlan-type dot1q 10
 description Gateway_for_VRF_CORE_1_VLAN10
 ip address 10.99.1.0 255.255.255.254
 service-manage ping permit
#
interface GigabitEthernet1/0/0.20
 vlan-type dot1q 20
 description Gateway_for_VRF_CORE_2_VLAN20
 ip address 10.99.2.0 255.255.255.254
 service-manage ping permit
#
interface GigabitEthernet1/0/0.30
 vlan-type dot1q 30
 description Gateway_for_VRF_CORE_3_VLAN30
 ip address 10.99.3.0 255.255.255.254
 service-manage ping permit
#
interface GigabitEthernet1/0/0.40
 vlan-type dot1q 40
 description Gateway_for_VRF_CORE_4_VLAN40
 ip address 10.99.4.0 255.255.255.254
 service-manage ping permit

bgp 65098
router id 10.99.245.252
 group blf01 external
 peer blf01 as-number 65099
 peer blf01 timer keepalive 3 hold 9
 peer 10.99.1.1 group blf01
 peer 10.99.2.1 group blf01
 peer 10.99.3.1 group blf01
 peer 10.99.4.1 group blf01
 group blf02 external
 peer blf02 as-number 65099
 peer blf02 timer keepalive 3 hold 9
 peer blf02 ebgp-max-hop 10
 peer 10.99.1.2 group blf02
 peer 10.99.2.2 group blf02
 peer 10.99.3.2 group blf02
 peer 10.99.4.2 group blf02


 ipv4-family unicast
  import-route static
  default-route imported
  peer blf01 enable
  peer blf02 enable
 

q
q
ip route-static 0.0.0.0 0 NULL 0

```
fw02
```bash

interface GigabitEthernet1/0/0
 description 99blf02-Et4
 undo shutdown
 service-manage ping permit
#
interface GigabitEthernet1/0/0.10
 vlan-type dot1q 10
 description Gateway_for_VRF_CORE_1_VLAN10
 ip address 10.99.1.3 255.255.255.254
 service-manage ping permit
#
interface GigabitEthernet1/0/0.20
 vlan-type dot1q 20
 description Gateway_for_VRF_CORE_2_VLAN20
 ip address 10.99.2.3 255.255.255.254
 service-manage ping permit
#
interface GigabitEthernet1/0/0.30
 vlan-type dot1q 30
 description Gateway_for_VRF_CORE_3_VLAN30
 ip address 10.99.3.3 255.255.255.254
 service-manage ping permit
#
interface GigabitEthernet1/0/0.40
 vlan-type dot1q 40
 description Gateway_for_VRF_CORE_4_VLAN40
 ip address 10.99.4.3 255.255.255.254
 service-manage ping permit

bgp 65098
router id 10.99.245.253
 group blf01 external
 peer blf01 as-number 65099
 peer blf01 timer keepalive 3 hold 9
 peer blf01 ebgp-max-hop 10
 peer 10.99.1.1 group blf01
 peer 10.99.2.1 group blf01
 peer 10.99.3.1 group blf01
 peer 10.99.4.1 group blf01
 group blf02 external
 peer blf02 as-number 65099
 peer blf02 timer keepalive 3 hold 9
 peer 10.99.1.2 group blf02
 peer 10.99.2.2 group blf02
 peer 10.99.3.2 group blf02
 peer 10.99.4.2 group blf02


 ipv4-family unicast
  import-route static
  default-route imported
  peer blf01 enable
  peer blf02 enable
 

q
q
ip route-static 0.0.0.0 0 NULL 0
```
</details>

fw1/2
Добавим наши сабинтерфейсы по разным зонам и сделаем правил для теста и широкое правило 10.99.0.0/21 на 10.99.0.0/21 чтоб поднялось bgp

<details>
<summary>Конфиг правил на fw </summary>

```bash
security-policy
 rule name Underlay
  source-address 10.99.0.0 21
  source-address 10.99.241.0 24
  destination-address 10.99.0.0 21
  destination-address 10.99.241.0 24
  action permit
 rule name VRF_10_30
  source-zone dmz
  destination-zone trust
  source-address 192.168.3.0 24
  destination-address 192.168.1.1 32
  action permit


firewall zone trust
 set priority 85
 add interface GigabitEthernet0/0/0
 add interface Eth-Trunk23
 add interface GigabitEthernet1/0/0.10
 add interface GigabitEthernet1/0/0
 add interface GigabitEthernet1/0/0.40

firewall zone dmz
 set priority 50
 add interface GigabitEthernet1/0/1
 add interface GigabitEthernet1/0/0.30
 add interface GigabitEthernet1/0/0.20

```
</details>

В целом все, теперь наши VRF получают дефолты от фаерволлов.
Плюс правилам мы разграничили определенные направления трафика между VRF
1. Permit 192.168.2.0/24 <-> 192.168.3.0/24
2. Permit 192.168.1.0/24 <-> 192.168.4.0/24
3. Permit Permit 192.168.3.0/24 -> 192.168.1.1/32
4. Deny any <-> any


### Общие конфиги
<details>
<summary>Конфиги </summary>

### 99-fw01 

<details>
<summary>FW01 </summary>

```bash
router id 10.99.245.252
#
undo telnet server enable
undo telnet ipv6 server enable
#
clock timezone UTC add 00:00:00
#
 hrp enable
 hrp interface Eth-Trunk23 remote 10.99.241.13
 hrp standby config enable
 hrp adjust bgp-cost enable 10
 hrp track interface Eth-Trunk23
#
manager-user admin
password cipher @%@%RqS9%jPme(=~B>N'q<F-QjAt,%-6KExumX]iVOO,O-}NjAwQ@%@%
service-type ssh
level 15
#
interface Eth-Trunk23
 description HRP LINK to 99-fw02
 ip address 10.99.241.12 255.255.255.254
 service-manage http permit
 service-manage https permit
 service-manage ping permit
 service-manage ssh permit
 service-manage snmp permit
 service-manage telnet permit
 service-manage netconf permit
#
interface GigabitEthernet0/0/0
 undo shutdown
 ip binding vpn-instance default
 ip address 10.99.245.252 255.255.255.0
 service-manage http permit
 service-manage https permit
 service-manage ping permit
 service-manage ssh permit
 service-manage snmp permit
 service-manage telnet permit
 service-manage netconf permit
#
interface GigabitEthernet1/0/0
 description 99blf01-Et4
 undo shutdown
 service-manage ping permit
#
interface GigabitEthernet1/0/0.10
 vlan-type dot1q 10
 description Gateway_for_VRF_CORE_1_VLAN10
 ip address 10.99.1.0 255.255.255.254
 service-manage ping permit
#
interface GigabitEthernet1/0/0.20
 vlan-type dot1q 20
 description Gateway_for_VRF_CORE_2_VLAN20
 ip address 10.99.2.0 255.255.255.254
 service-manage ping permit
#
interface GigabitEthernet1/0/0.30
 vlan-type dot1q 30
 description Gateway_for_VRF_CORE_3_VLAN30
 ip address 10.99.3.0 255.255.255.254
 service-manage ping permit
#
interface GigabitEthernet1/0/0.40
 vlan-type dot1q 40
 description Gateway_for_VRF_CORE_4_VLAN40
 ip address 10.99.4.0 255.255.255.254
 service-manage ping permit
#
interface GigabitEthernet1/0/1
 description 99blf02-Et4
 undo shutdown
#
interface GigabitEthernet1/0/2
 description HRP LINK to 99-fw02
 undo shutdown
 eth-trunk 23
#
interface GigabitEthernet1/0/3
 description HRP LINK to 99-fw02
 undo shutdown
 eth-trunk 23
#
interface GigabitEthernet1/0/4
 undo shutdown
#
interface Virtual-if0
#
interface NULL0
#
firewall zone local
 set priority 100
#
firewall zone trust
 set priority 85
 add interface GigabitEthernet0/0/0
 add interface Eth-Trunk23
 add interface GigabitEthernet1/0/0.10
 add interface GigabitEthernet1/0/0
 add interface GigabitEthernet1/0/0.40
#
firewall zone untrust
 set priority 5
#
firewall zone dmz
 set priority 50
 add interface GigabitEthernet1/0/1
 add interface GigabitEthernet1/0/0.30
 add interface GigabitEthernet1/0/0.20

bgp 65098
 group blf01 external
 peer blf01 as-number 65099
 peer blf01 timer keepalive 3 hold 9
 peer 10.99.1.1 as-number 65099
 peer 10.99.1.1 group blf01
 peer 10.99.2.1 as-number 65099
 peer 10.99.2.1 group blf01
 peer 10.99.3.1 as-number 65099
 peer 10.99.3.1 group blf01
 peer 10.99.4.1 as-number 65099
 peer 10.99.4.1 group blf01
 group blf02 external
 peer blf02 as-number 65099
 peer blf02 ebgp-max-hop 10
 peer blf02 timer keepalive 3 hold 9
 peer 10.99.1.2 as-number 65099
 peer 10.99.1.2 group blf02
 peer 10.99.2.2 as-number 65099
 peer 10.99.2.2 group blf02
 peer 10.99.3.2 as-number 65099
 peer 10.99.3.2 group blf02
 peer 10.99.4.2 as-number 65099
 peer 10.99.4.2 group blf02
 #
 ipv4-family unicast
  undo synchronization
  default-route imported
  import-route static
  peer blf01 enable
  peer 10.99.1.1 enable
  peer 10.99.1.1 group blf01
  peer 10.99.2.1 enable
  peer 10.99.2.1 group blf01
  peer 10.99.3.1 enable
  peer 10.99.3.1 group blf01
  peer 10.99.4.1 enable
  peer 10.99.4.1 group blf01
  peer blf02 enable
  peer 10.99.1.2 enable
  peer 10.99.1.2 group blf02
  peer 10.99.2.2 enable
  peer 10.99.2.2 group blf02
  peer 10.99.3.2 enable
  peer 10.99.3.2 group blf02
  peer 10.99.4.2 enable
  peer 10.99.4.2 group blf02
#
ip route-static 0.0.0.0 0.0.0.0 NULL0
#
undo ssh server compatible-ssh1x enable
stelnet server enable
ssh authentication-type default password
#
user-interface con 0
 authentication-mode aaa
user-interface vty 0 4
 authentication-mode aaa
 protocol inbound ssh
user-interface vty 16 20

security-policy
 rule name Underlay
  source-address 10.99.0.0 21
  source-address 10.99.241.0 24
  destination-address 10.99.0.0 21
  destination-address 10.99.241.0 24
  action permit
 rule name VRF_10_30
  source-zone dmz
  destination-zone trust
  source-address 192.168.3.0 24
  destination-address 192.168.1.1 32
  action permit
#

 ```
 </details>

 ### 99-fw02 
<details>
<summary>FW02 </summary>

```bash
router id 10.99.245.253
#
 hrp enable
 hrp standby-device
 hrp interface Eth-Trunk23 remote 10.99.241.12
 hrp standby config enable
 hrp adjust bgp-cost enable 20
 hrp track interface Eth-Trunk23
#
ip vpn-instance default
 ipv4-family

#
interface Eth-Trunk23
 description HRP LINK to 99-fw01
 ip address 10.99.241.13 255.255.255.254
 service-manage http permit
 service-manage https permit
 service-manage ping permit
 service-manage ssh permit
 service-manage snmp permit
 service-manage telnet permit
 service-manage netconf permit
#
interface GigabitEthernet0/0/0
 undo shutdown
 ip binding vpn-instance default
 ip address 10.99.245.253 255.255.255.0
 service-manage http permit
 service-manage https permit
 service-manage ping permit
 service-manage ssh permit
 service-manage snmp permit
 service-manage telnet permit
 service-manage netconf permit
#
interface GigabitEthernet1/0/0
 description 99blf02-Et4
 undo shutdown
 ip address 10.99.242.9 255.255.255.254
 service-manage ping permit
#
interface GigabitEthernet1/0/0.10
 vlan-type dot1q 10
 description Gateway_for_VRF_CORE_1_VLAN10
 ip address 10.99.1.3 255.255.255.254
 service-manage ping permit
#
interface GigabitEthernet1/0/0.20
 vlan-type dot1q 20
 description Gateway_for_VRF_CORE_2_VLAN20
 ip address 10.99.2.3 255.255.255.254
 service-manage ping permit
#
interface GigabitEthernet1/0/0.30
 vlan-type dot1q 30
 description Gateway_for_VRF_CORE_3_VLAN30
 ip address 10.99.3.3 255.255.255.254
 service-manage ping permit
#
interface GigabitEthernet1/0/0.40
 vlan-type dot1q 40
 description Gateway_for_VRF_CORE_4_VLAN40
 ip address 10.99.4.3 255.255.255.254
 service-manage ping permit
#
interface GigabitEthernet1/0/1
 undo shutdown
 ip address 10.99.242.11 255.255.255.254
#
interface GigabitEthernet1/0/2
 description HRP LINK to 99-fw01
 undo shutdown
 eth-trunk 23
#
interface GigabitEthernet1/0/3
 description HRP LINK to 99-fw01
 undo shutdown
 eth-trunk 23
#
interface GigabitEthernet1/0/4
 undo shutdown
#
interface Virtual-if0
#
interface NULL0
#
firewall zone local
 set priority 100
#
firewall zone trust
 set priority 85
 add interface GigabitEthernet0/0/0
 add interface Eth-Trunk23
 add interface GigabitEthernet1/0/0.10
 add interface GigabitEthernet1/0/0
 add interface GigabitEthernet1/0/0.40
#
firewall zone untrust
 set priority 5
#
firewall zone dmz
 set priority 50
 add interface GigabitEthernet1/0/1
 add interface GigabitEthernet1/0/0.30
 add interface GigabitEthernet1/0/0.20
#
l2tp-group default-lns
#
bgp 65098
 group blf01 external
 peer blf01 as-number 65099
 peer blf01 ebgp-max-hop 10
 peer blf01 timer keepalive 3 hold 9
 peer 10.99.1.1 as-number 65099
 peer 10.99.1.1 group blf01
 peer 10.99.2.1 as-number 65099
 peer 10.99.2.1 group blf01
 peer 10.99.3.1 as-number 65099
 peer 10.99.3.1 group blf01
 peer 10.99.4.1 as-number 65099
 peer 10.99.4.1 group blf01
 group blf02 external
 peer blf02 as-number 65099
 peer blf02 timer keepalive 3 hold 9
 peer 10.99.1.2 as-number 65099
 peer 10.99.1.2 group blf02
 peer 10.99.2.2 as-number 65099
 peer 10.99.2.2 group blf02
 peer 10.99.3.2 as-number 65099
 peer 10.99.3.2 group blf02
 peer 10.99.4.2 as-number 65099
 peer 10.99.4.2 group blf02
 #
 ipv4-family unicast
  undo synchronization
  default-route imported
  import-route static
  peer blf01 enable
  peer 10.99.1.1 enable
  peer 10.99.1.1 group blf01
  peer 10.99.2.1 enable
  peer 10.99.2.1 group blf01
  peer 10.99.3.1 enable
  peer 10.99.3.1 group blf01
  peer 10.99.4.1 enable
  peer 10.99.4.1 group blf01
  peer blf02 enable
  peer 10.99.1.2 enable
  peer 10.99.1.2 group blf02
  peer 10.99.2.2 enable
  peer 10.99.2.2 group blf02
  peer 10.99.3.2 enable
  peer 10.99.3.2 group blf02
  peer 10.99.4.2 enable
  peer 10.99.4.2 group blf02
#
ip route-static 0.0.0.0 0.0.0.0 NULL0
#
security-policy
 rule name Underlay
  source-address 10.99.0.0 21
  source-address 10.99.241.0 24
  source-address 10.99.242.0 24
  destination-address 10.99.0.0 21
  destination-address 10.99.241.0 24
  destination-address 10.99.242.0 24
  action permit
 rule name VRF_10_30
  source-zone dmz
  destination-zone trust
  source-address 192.168.3.0 24
  destination-address 192.168.1.1 32
  action permit

 ```
</details>

### 99-blf1 (Border Leaf 1)
```bash

 ```

 ### 99-blf2 (Border Leaf 2)
 ```bash

 ```
### 99-lf3 (Leaf 3)
```bash


 ```

 ### 99-lf4 (Leaf 4)
```bash



 ```

 ### 99-sp1 (Spine 1)
 ```bash
configure terminal
hostname 99-sp1

ip routing
service routing protocols model multi-agent

vrf instance MGMT
 ip routing 

interface Management 1
vrf MGMT
ip address 10.99.245.11/24

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

vrf instance MGMT
 ip routing 

interface Management 1
vrf MGMT
ip address 10.99.245.22/24

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
vlan N0

interface Ethernet1
   description to-99-lf4-Eth3
   switchport mode trunk
   switchport trunk allowed vlan 10,20
   mtu 9100
   no shutdown


interface Vlan10
   description VMKernel-VLAN10
   ip address 192.168.1.N/24
   mtu 9100
   no shutdown


ip route 0.0.0.0/0 192.168.1.254

 ```
### 99-esx2 (ESX 2)
```bash
 hostname 99-esx2
 spanning-tree mode mstp
 vlan 10,20
 vrf instance VRF_CORE_1
 vrf instance VRF_CORE_2

interface Ethernet1
   description to-99-blf2-Eth3
   mtu 9100
   switchport trunk allowed vlan 10,20
   switchport mode trunk

interface Ethernet2
   description to-99-lf3-Eth4
   mtu 9100
   switchport trunk allowed vlan 10,20
   switchport mode trunk


interface Vlan10
   description VMKernel-VLAN10
   mtu 9100
   vrf VRF_CORE_1
   ip address 192.168.1.2/24

interface Vlan20
   description VMKernel-VLAN20
   mtu 9100
   vrf VRF_CORE_2
   ip address 192.168.2.1/24

ip routing
ip routing vrf VRF_CORE_1
ip routing vrf VRF_CORE_2

ip route vrf VRF_CORE_1 0.0.0.0/0 192.168.1.254
ip route vrf VRF_CORE_2 0.0.0.0/0 192.168.2.254
```
</details>

---

## Проверка IP связности
 ## 1. Проверка настроенного mlag домена и портченнелов blf1/2
 ```
 99-blf1#sh mlag
MLAG Configuration:
domain-id                          :                  12
local-interface                    :            Vlan4094
peer-address                       :         10.99.246.2
peer-link                          :      Port-Channel78
hb-peer-address                    :         10.99.245.2
hb-peer-vrf                        :                MGMT
peer-config                        :          consistent

MLAG Status:
state                              :              Active
negotiation status                 :           Connected
peer-link status                   :                  Up
local-int status                   :                  Up
system-id                          :   52:00:00:cb:38:c2
dual-primary detection             :          Configured
dual-primary interface errdisabled :               False

MLAG Ports:
Disabled                           :                   0
Configured                         :                   0
Inactive                           :                   0
Active-partial                     :                   1
```
```
99-blf1#sh lacp peer
State: A = Active, P = Passive; S=ShortTimeout, L=LongTimeout;
       G = Aggregable, I = Individual; s+=InSync, s-=OutOfSync;
       C = Collecting, X = state machine expired,
       D = Distributing, d = default neighbor state
                 |                        Partner
 Port    Status  | Sys-id                    Port#   State     OperKey  PortPri
------ ----------|------------------------- ------- --------- --------- -------
Port Channel Port-Channel3*:
 Et3     Bundled | 8000,50-00-00-d5-5d-c0        1   ALGs+CD    0x0001    32768
Port Channel Port-Channel5*:
 Et5     Bundled | 8000,50-00-00-6b-2e-70        2   ALGs+CD    0x0002    32768
Port Channel Port-Channel78:
 Et7     Bundled | 8000,50-00-00-cb-38-c2        7   ALGs+CD    0x004e    32768
 Et8     Bundled | 8000,50-00-00-cb-38-c2        8   ALGs+CD    0x004e    32768


```
```
99-lf3#sh lacp peer
State: A = Active, P = Passive; S=ShortTimeout, L=LongTimeout;
       G = Aggregable, I = Individual; s+=InSync, s-=OutOfSync;
       C = Collecting, X = state machine expired,
       D = Distributing, d = default neighbor state
                 |                        Partner
 Port    Status  | Sys-id                    Port#   State     OperKey  PortPri
------ ----------|------------------------- ------- --------- --------- -------
Port Channel Port-Channel3:
 Et3     Bundled | 8000,50-00-00-1b-5e-8d        1   ALGs+CD    0x0001    32768
Port Channel Port-Channel4:
 Et4     Bundled | 8000,50-00-00-03-37-66        1   ALGs+CD    0x0001    32768
```

```
99-blf1#sh mac address-table vl 20
          Mac Address Table
------------------------------------------------------------------

Vlan    Mac Address       Type        Ports      Moves   Last Move
----    -----------       ----        -----      -----   ---------
  20    0000.0000.0001    STATIC      Cpu
  20    5000.006b.2e70    DYNAMIC     Po5        1       0:09:03 ago
  20    5000.00cb.38c2    STATIC      Po78
Total Mac Addresses for this criterion: 3

99-blf1#sh mac address-table vl 10
          Mac Address Table
------------------------------------------------------------------

Vlan    Mac Address       Type        Ports      Moves   Last Move
----    -----------       ----        -----      -----   ---------
  10    0000.0000.0001    STATIC      Cpu
  10    5000.00cb.38c2    STATIC      Po78
Total Mac Addresses for this criterion: 2


```

## 2.  Проверка таблицы bgp VRF_CORE_1/2  
```
99-blf2#sh ip bgp vrf VRF_CORE_1
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  AIGP       LocPref Weight  Path
 * >      192.168.1.0/24         -                     -       -          -       0       i
          192.168.1.0/24         10.99.244.1           0       -          100     0       i Or-ID: 10.99.243.1 C-LST: 10.99.243.11
          192.168.1.1/32         10.99.244.1           0       -          100     0       i Or-ID: 10.99.243.1 C-LST: 10.99.243.22
          192.168.1.1/32         10.99.244.1           0       -          100     0       i Or-ID: 10.99.243.1 C-LST: 10.99.243.11
          192.168.1.2/32         10.99.244.1           0       -          100     0       i Or-ID: 10.99.243.1 C-LST: 10.99.243.11
          192.168.1.2/32         10.99.244.1           0       -          100     0       i Or-ID: 10.99.243.1 C-LST: 10.99.243.22
 * >Ec    192.168.4.0/24         10.99.244.4           0       -          100     0       i Or-ID: 10.99.243.4 C-LST: 10.99.243.22
 *  ec    192.168.4.0/24         10.99.244.4           0       -          100     0       i Or-ID: 10.99.243.4 C-LST: 10.99.243.11
 * >Ec    192.168.4.1/32         10.99.244.4           0       -          100     0       i Or-ID: 10.99.243.4 C-LST: 10.99.243.22
 *  ec    192.168.4.1/32         10.99.244.3           0       -          100     0       i Or-ID: 10.99.243.3 C-LST: 10.99.243.22
 *  ec    192.168.4.1/32         10.99.244.3           0       -          100     0       i Or-ID: 10.99.243.3 C-LST: 10.99.243.11
 *  ec    192.168.4.1/32         10.99.244.4           0       -          100     0       i Or-ID: 10.99.243.4 C-LST: 10.99.243.11


```

```
9sh ip bgp vrf VRF_CORE_2
BGP routing table information for VRF VRF_CORE_2
Router identifier 192.168.2.254, local AS number 65099
Route status codes: s - suppressed, * - valid, > - active, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup, L - labeled-unicast
                    % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI Origin Validation codes: V - valid, I - invalid, U - unknown
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  AIGP       LocPref Weight  Path
 * >      192.168.2.0/24         -                     -       -          -       0       i
          192.168.2.1/32         10.99.244.1           0       -          100     0       i Or-ID: 10.99.243.1 C-LST: 10.99.243.22
          192.168.2.1/32         10.99.244.1           0       -          100     0       i Or-ID: 10.99.243.1 C-LST: 10.99.243.11
 * >Ec    192.168.3.0/24         10.99.244.4           0       -          100     0       i Or-ID: 10.99.243.4 C-LST: 10.99.243.22
 *  ec    192.168.3.0/24         10.99.244.4           0       -          100     0       i Or-ID: 10.99.243.4 C-LST: 10.99.243.11
 * >Ec    192.168.3.1/32         10.99.244.4           0       -          100     0       i Or-ID: 10.99.243.4 C-LST: 10.99.243.22
 *  ec    192.168.3.1/32         10.99.244.3           0       -          100     0       i Or-ID: 10.99.243.3 C-LST: 10.99.243.22
 *  ec    192.168.3.1/32         10.99.244.4           0       -          100     0       i Or-ID: 10.99.243.4 C-LST: 10.99.243.11
 *  ec    192.168.3.1/32         10.99.244.3           0       -          100     0       i Or-ID: 10.99.243.3 C-LST: 10.99.243.11


```
Видим что каждый маршрут со стороны сети 192.168.3.0/24 и 192.168.4.0/24  имеет по два пути через каждый спайн с двумя разными vtep так как там настроен multihoming


```
99-lf3#sh ip bgp vrf VRF_CORE_1
BGP routing table information for VRF VRF_CORE_1
Router identifier 192.168.4.254, local AS number 65099

          Network                Next Hop              Metric  AIGP       LocPref Weight  Path
 * >Ec    192.168.1.0/24         10.99.244.1           0       -          100     0       i Or-ID: 10.99.243.2 C-LST: 10.99.243.22
 *  ec    192.168.1.0/24         10.99.244.1           0       -          100     0       i Or-ID: 10.99.243.1 C-LST: 10.99.243.11
 * >Ec    192.168.1.1/32         10.99.244.1           0       -          100     0       i Or-ID: 10.99.243.2 C-LST: 10.99.243.11
 *  ec    192.168.1.1/32         10.99.244.1           0       -          100     0       i Or-ID: 10.99.243.2 C-LST: 10.99.243.22
 *  ec    192.168.1.1/32         10.99.244.1           0       -          100     0       i Or-ID: 10.99.243.1 C-LST: 10.99.243.11
 *  ec    192.168.1.1/32         10.99.244.1           0       -          100     0       i Or-ID: 10.99.243.1 C-LST: 10.99.243.22
 * >      192.168.4.0/24         -                     -       -          -       0       i
 *        192.168.4.0/24         10.99.244.4           0       -          100     0       i Or-ID: 10.99.243.4 C-LST: 10.99.243.22
          192.168.4.1/32         10.99.244.4           0       -          100     0       i Or-ID: 10.99.243.4 C-LST: 10.99.243.22
          192.168.4.1/32         10.99.244.4           0       -          100     0       i Or-ID: 10.99.243.4 C-LST: 10.99.243.11
99-lf3#
99-lf3#sh ip bgp vrf VRF_CORE_2
BGP routing table information for VRF VRF_CORE_2
Router identifier 192.168.3.254, local AS number 65099

          Network                Next Hop              Metric  AIGP       LocPref Weight  Path
 * >Ec    192.168.2.0/24         10.99.244.1           0       -          100     0       i Or-ID: 10.99.243.2 C-LST: 10.99.243.22
 *  ec    192.168.2.0/24         10.99.244.1           0       -          100     0       i Or-ID: 10.99.243.2 C-LST: 10.99.243.11
 * >Ec    192.168.2.1/32         10.99.244.1           0       -          100     0       i Or-ID: 10.99.243.1 C-LST: 10.99.243.22
 *  ec    192.168.2.1/32         10.99.244.1           0       -          100     0       i Or-ID: 10.99.243.1 C-LST: 10.99.243.11
 *  ec    192.168.2.1/32         10.99.244.1           0       -          100     0       i Or-ID: 10.99.243.2 C-LST: 10.99.243.11
 *  ec    192.168.2.1/32         10.99.244.1           0       -          100     0       i Or-ID: 10.99.243.2 C-LST: 10.99.243.22
 * >      192.168.3.0/24         -                     -       -          -       0       i
 *        192.168.3.0/24         10.99.244.4           0       -          100     0       i Or-ID: 10.99.243.4 C-LST: 10.99.243.22
          192.168.3.1/32         10.99.244.4           0       -          100     0       i Or-ID: 10.99.243.4 C-LST: 10.99.243.22
          192.168.3.1/32         10.99.244.4           0       -          100     0       i Or-ID: 10.99.243.4 C-LST: 10.99.243.11
```
Видим что каждый маршрут со стороны сети 192.168.1.0/24 и 192.168.2.0/24  имеет по два пути через каждый спайн с двумя одинаковыми vtep но разными Or-ID  потому что blf1/2 собраны в mlag 
Также видим прилетающие роут тайп 1 и 4 маршруты с лифов 3/4 свидетельствующие об использовании multihoming
```
99-blf1#sh bgp evpn route-type auto-discovery
BGP routing table information for VRF default
Router identifier 10.99.243.1, local AS number 65099
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.99.243.3:30 auto-discovery 0 0000:0000:0000:0000:3403
                                 10.99.244.3           -       100     0       i Or-ID: 10.99.243.3 C-LST: 10.99.243.22
 *  ec    RD: 10.99.243.3:30 auto-discovery 0 0000:0000:0000:0000:3403
                                 10.99.244.3           -       100     0       i Or-ID: 10.99.243.3 C-LST: 10.99.243.11
 * >Ec    RD: 10.99.243.4:30 auto-discovery 0 0000:0000:0000:0000:3403
                                 10.99.244.4           -       100     0       i Or-ID: 10.99.243.4 C-LST: 10.99.243.22
 *  ec    RD: 10.99.243.4:30 auto-discovery 0 0000:0000:0000:0000:3403
                                 10.99.244.4           -       100     0       i Or-ID: 10.99.243.4 C-LST: 10.99.243.11
 * >Ec    RD: 10.99.244.3:1 auto-discovery 0000:0000:0000:0000:3403
                                 10.99.244.3           -       100     0       i Or-ID: 10.99.243.3 C-LST: 10.99.243.22
 *  ec    RD: 10.99.244.3:1 auto-discovery 0000:0000:0000:0000:3403
                                 10.99.244.3           -       100     0       i Or-ID: 10.99.243.3 C-LST: 10.99.243.11
 * >Ec    RD: 10.99.244.4:1 auto-discovery 0000:0000:0000:0000:3403
                                 10.99.244.4           -       100     0       i Or-ID: 10.99.243.4 C-LST: 10.99.243.22
 *  ec    RD: 10.99.244.4:1 auto-discovery 0000:0000:0000:0000:3403
                                 10.99.244.4           -       100     0       i Or-ID: 10.99.243.4 C-LST: 10.99.243.11
 * >Ec    RD: 10.99.243.3:40 auto-discovery 0 0000:0000:0000:0000:3404
                                 10.99.244.3           -       100     0       i Or-ID: 10.99.243.3 C-LST: 10.99.243.22
 *  ec    RD: 10.99.243.3:40 auto-discovery 0 0000:0000:0000:0000:3404
                                 10.99.244.3           -       100     0       i Or-ID: 10.99.243.3 C-LST: 10.99.243.11
 * >Ec    RD: 10.99.243.4:40 auto-discovery 0 0000:0000:0000:0000:3404
                                 10.99.244.4           -       100     0       i Or-ID: 10.99.243.4 C-LST: 10.99.243.22
 *  ec    RD: 10.99.243.4:40 auto-discovery 0 0000:0000:0000:0000:3404
                                 10.99.244.4           -       100     0       i Or-ID: 10.99.243.4 C-LST: 10.99.243.11
 * >Ec    RD: 10.99.244.3:1 auto-discovery 0000:0000:0000:0000:3404
                                 10.99.244.3           -       100     0       i Or-ID: 10.99.243.3 C-LST: 10.99.243.22
 *  ec    RD: 10.99.244.3:1 auto-discovery 0000:0000:0000:0000:3404
                                 10.99.244.3           -       100     0       i Or-ID: 10.99.243.3 C-LST: 10.99.243.11
 * >Ec    RD: 10.99.244.4:1 auto-discovery 0000:0000:0000:0000:3404
                                 10.99.244.4           -       100     0       i Or-ID: 10.99.243.4 C-LST: 10.99.243.22
 *  ec    RD: 10.99.244.4:1 auto-discovery 0000:0000:0000:0000:3404
                                 10.99.244.4           -       100     0       i Or-ID: 10.99.243.4 C-LST: 10.99.243.11
99-blf1#
99-blf1#sh bgp evpn route-type ethernet-segment
BGP routing table information for VRF default
Router identifier 10.99.243.1, local AS number 65099
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.99.244.3:1 ethernet-segment 0000:0000:0000:0000:3403 10.99.244.3
                                 10.99.244.3           -       100     0       i Or-ID: 10.99.243.3 C-LST: 10.99.243.22
 *  ec    RD: 10.99.244.3:1 ethernet-segment 0000:0000:0000:0000:3403 10.99.244.3
                                 10.99.244.3           -       100     0       i Or-ID: 10.99.243.3 C-LST: 10.99.243.11
 * >Ec    RD: 10.99.244.4:1 ethernet-segment 0000:0000:0000:0000:3403 10.99.244.4
                                 10.99.244.4           -       100     0       i Or-ID: 10.99.243.4 C-LST: 10.99.243.22
 *  ec    RD: 10.99.244.4:1 ethernet-segment 0000:0000:0000:0000:3403 10.99.244.4
                                 10.99.244.4           -       100     0       i Or-ID: 10.99.243.4 C-LST: 10.99.243.11
 * >Ec    RD: 10.99.244.3:1 ethernet-segment 0000:0000:0000:0000:3404 10.99.244.3
                                 10.99.244.3           -       100     0       i Or-ID: 10.99.243.3 C-LST: 10.99.243.22
 *  ec    RD: 10.99.244.3:1 ethernet-segment 0000:0000:0000:0000:3404 10.99.244.3
                                 10.99.244.3           -       100     0       i Or-ID: 10.99.243.3 C-LST: 10.99.243.11
 * >Ec    RD: 10.99.244.4:1 ethernet-segment 0000:0000:0000:0000:3404 10.99.244.4
                                 10.99.244.4           -       100     0       i Or-ID: 10.99.243.4 C-LST: 10.99.243.22
 *  ec    RD: 10.99.244.4:1 ethernet-segment 0000:0000:0000:0000:3404 10.99.244.4
                                 10.99.244.4           -       100     0       i Or-ID: 10.99.243.4 C-LST: 10.99.243.11
```


## 3. Проверка межсерверной связности между VM 
Пингуем с ESXI2 , который живет в двух VRF 
```
99-esx2#ping vrf VRF_CORE_1 192.168.1.1
PING 192.168.1.1 (192.168.1.1) 72(100) bytes of data.
80 bytes from 192.168.1.1: icmp_seq=1 ttl=64 time=125 ms
80 bytes from 192.168.1.1: icmp_seq=2 ttl=64 time=118 ms
80 bytes from 192.168.1.1: icmp_seq=3 ttl=64 time=112 ms
80 bytes from 192.168.1.1: icmp_seq=4 ttl=64 time=107 ms
80 bytes from 192.168.1.1: icmp_seq=5 ttl=64 time=100 ms

--- 192.168.1.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 47ms
rtt min/avg/max/mdev = 100.337/113.012/125.676/8.781 ms, pipe 5, ipg/ewma 11.800/118.707 ms
99-esx2#
99-esx2#ping vrf VRF_CORE_1 192.168.4.1
PING 192.168.4.1 (192.168.4.1) 72(100) bytes of data.
80 bytes from 192.168.4.1: icmp_seq=1 ttl=62 time=67.2 ms
80 bytes from 192.168.4.1: icmp_seq=2 ttl=62 time=58.1 ms
80 bytes from 192.168.4.1: icmp_seq=3 ttl=62 time=49.1 ms
80 bytes from 192.168.4.1: icmp_seq=4 ttl=62 time=41.7 ms
80 bytes from 192.168.4.1: icmp_seq=5 ttl=62 time=40.6 ms

--- 192.168.4.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 44ms
rtt min/avg/max/mdev = 40.621/51.376/67.227/10.113 ms, pipe 5, ipg/ewma 11.115/58.622 ms
99-esx2#
99-esx2#ping vrf VRF_CORE_2 192.168.3.1
PING 192.168.3.1 (192.168.3.1) 72(100) bytes of data.
80 bytes from 192.168.3.1: icmp_seq=1 ttl=62 time=35.7 ms
80 bytes from 192.168.3.1: icmp_seq=2 ttl=62 time=25.6 ms
80 bytes from 192.168.3.1: icmp_seq=3 ttl=62 time=22.2 ms
80 bytes from 192.168.3.1: icmp_seq=4 ttl=62 time=22.8 ms
80 bytes from 192.168.3.1: icmp_seq=5 ttl=62 time=26.3 ms

--- 192.168.3.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 67ms
rtt min/avg/max/mdev = 22.230/26.551/35.722/4.850 ms, pipe 4, ipg/ewma 16.879/31.003 ms
99-esx2#
99-esx2#ping vrf VRF_CORE_2 192.168.3.254
PING 192.168.3.254 (192.168.3.254) 72(100) bytes of data.
80 bytes from 192.168.3.254: icmp_seq=1 ttl=63 time=47.7 ms
80 bytes from 192.168.3.254: icmp_seq=2 ttl=63 time=38.2 ms
80 bytes from 192.168.3.254: icmp_seq=3 ttl=63 time=29.5 ms
80 bytes from 192.168.3.254: icmp_seq=4 ttl=63 time=21.8 ms
80 bytes from 192.168.3.254: icmp_seq=5 ttl=63 time=20.3 ms

--- 192.168.3.254 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 43ms
rtt min/avg/max/mdev = 20.358/31.563/47.787/10.318 ms, pipe 5, ipg/ewma 10.910/38.979 ms
99-esx2#
99-esx2#ping vrf VRF_CORE_1 192.168.4.254
PING 192.168.4.254 (192.168.4.254) 72(100) bytes of data.
80 bytes from 192.168.4.254: icmp_seq=1 ttl=63 time=35.3 ms
80 bytes from 192.168.4.254: icmp_seq=2 ttl=63 time=25.7 ms
80 bytes from 192.168.4.254: icmp_seq=3 ttl=63 time=23.0 ms
80 bytes from 192.168.4.254: icmp_seq=4 ttl=63 time=20.5 ms
80 bytes from 192.168.4.254: icmp_seq=5 ttl=63 time=13.8 ms

--- 192.168.4.254 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 65ms
rtt min/avg/max/mdev = 13.822/23.719/35.339/7.039 ms, pipe 4, ipg/ewma 16.375/29.060 ms
99-esx2#


```
## 4. Проверка межсерверной связности между VM из разных VRF 

```
99-esx4#
99-esx4#ping 192.168.2.1
PING 192.168.2.1 (192.168.2.1) 72(100) bytes of data.
From 192.168.4.254 icmp_seq=1 Destination Net Unreachable

--- 192.168.2.1 ping statistics ---
5 packets transmitted, 0 received, +1 errors, 100% packet loss, time 52ms
pipe 2
99-esx4#
99-esx4#ping 192.168.3.1
PING 192.168.3.1 (192.168.3.1) 72(100) bytes of data.
From 192.168.4.254 icmp_seq=1 Destination Net Unreachable

--- 192.168.3.1 ping statistics ---
5 packets transmitted, 0 received, +1 errors, 100% packet loss, time 37ms
```
Пинг отсутствует по скольку VRF изолированы друг от друга и esx-2 не выступает в виде Роутера объединяющего VRF  


