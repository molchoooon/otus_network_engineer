# Проектная работа:  Реализовать связность двух ЦОД с применением дизайна Clos topology Multisite

## Задание
1. Настроить 2 POD POD-99 POD-199 
2. Настроить DCI связность
3. Сделать один растянутый влан под требования заказчика для L2 связности ВМ Заказчика между подами ( Vlan 99)
4. Настроить два  VRF в POD -199 - VRF13(изолированный, растянутый на оба цода), VRF14 - затерминировать VRF14 на фаерволлах POD99 для связности с VRF 1 и 4 в  POD99


---

## Топология сети
![Схема Сети ](fabric_scheme.jpg)


## IP-план (Address Plan) 
## POD-99
<details>
<summary>IP Plan </summary>

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

| ESXi Device | Физические порты (Trunk) | Виртуальные интерфейсы (SVI)| IP Address/Маска | Gateway     |
|-------------|--------------------------|---------------------------|------------------|---------------|
| 99-esx1     | Eth1 → 99-blf1 Eth3   Po1| Vlan10 (SVI для VLAN10)   | 192.168.1.1/24   | 192.168.1.254 |
| 99-esx1     | Eth2 → 99-blf2 Eth3   Po1| Vlan10 (SVI для VLAN10)   | 192.168.1.1/24   | 192.168.1.254 |
| 99-esx2     | Eth1 → 99-blf1 Eth5   Po2| Vlan10 (SVI для VLAN10)   | 192.168.1.2/24   | 192.168.1.254 |
| 99-esx2     | Eth1 → 99-blf2 Eth5   Po2| Vlan20 (SVI для VLAN20)   | 192.168.2.1/24   | 192.168.2.254 |
| 99-esx2     | Eth1 → 99-blf1 Eth5   Po2| Vlan10 (SVI для VLAN10)   | 192.168.1.2/24   | 192.168.1.254 |
| 99-esx2     | Eth1 → 99-blf2 Eth5   Po2| Vlan20 (SVI для VLAN20)   | 192.168.2.1/24   | 192.168.2.254 |
| 99-esx3     | Eth1 → 99-lf3 Eth3    Po1| Vlan30 (SVI для VLAN30)   | 192.168.3.1/24   | 192.168.3.254 |
| 99-esx3     | Eth1 → 99-lf4 Eth3    Po1| Vlan30 (SVI для VLAN30)   | 192.168.3.1/24   | 192.168.3.254 |
| 99-esx4     | Eth1 → 99-lf3 Eth4    Po1| Vlan40 (SVI для VLAN40)   | 192.168.4.1/24   | 192.168.4.254 |
| 99-esx4     | Eth1 → 99-lf4 Eth4    Po1| Vlan40 (SVI для VLAN40)   | 192.168.4.1/24   | 192.168.4.254 |
| 99-esx3     | Eth1 → 99-lf3 Eth3    Po1| Vlan99 (SVI для VLAN99)   | 192.168.99.5/24  | -             |
| 99-esx3     | Eth1 → 99-lf4 Eth3    Po1| Vlan99 (SVI для VLAN99)   | 192.168.99.6/24  | -             |
| 99-esx4     | Eth1 → 99-lf3 Eth4    Po1| Vlan99 (SVI для VLAN99)   | 192.168.99.5/24  | -             |
| 99-esx4     | Eth1 → 99-lf4 Eth4    Po1| Vlan99 (SVI для VLAN99)   | 192.168.99.6/24  | -             |

</details>

## POD-199
<details>
<summary>IP Plan </summary>

### Underlay сеть (Fabric Links - Point-to-Point /31 из сети 10.199.241.0/24 10.199.242.0/24)

| Device Name | IP Address/Маска | Port            | Remote Device | Remote Port | Description         |
|-------------|------------------|-----------------|---------------|-------------|---------------------|
| 199-lf01    | 10.199.241.0/31  | Ethernet1/1     | 199-sp01      | Ethernet1/1 | to Spine1           |
| 199-lf01    | 10.199.242.0/31  | Ethernet1/2     | 199-sp02      | Ethernet1/1 | to Spine2           |
| 199-lf01    | -                | Ethernet1/3 Po34| 199-lf02      | Ethernet1/3 | 199-lf02 Po34       |
| 199-lf01    | -                | Ethernet1/4 Po34| 199-lf02      | Ethernet1/4 | 199-lf02 Po34       |
| 199-lf01    | -                | Ethernet1/5 Po5 | 199-esx1      | Ethernet1   | 199-esx1 et1        |
| 199-lf01    | -                | Ethernet1/6 Po6 | 199-esx2      | Ethernet2   | 199-esx2 et1        | 
| 199-lf02    | 10.199.241.2/31  | Ethernet1/1     | 199-sp01      | Ethernet1/2 | to Spine1           |
| 199-lf02    | 10.199.242.2/31  | Ethernet1/2     | 199-sp02      | Ethernet1/2 | to Spine2           |
| 199-lf02    | -                | Ethernet1/3 Po34| 199-lf01      | Ethernet1/3 | 199-lf01 Po34       |
| 199-lf02    | -                | Ethernet1/4 Po34| 199-lf01      | Ethernet1/4 | 199-lf01 Po34       |
| 199-lf02    | -                | Ethernet1/5 Po5 | 199-esx1      | Ethernet1   | 199-esx1 et1        |
| 199-lf02    | -                | Ethernet1/6 Po6 | 199-esx2      | Ethernet2   | 199-esx2 et1        | 
| 199-bgw1    | 10.199.241.4/31  | Ethernet1       | 199-sp01      | Ethernet1/3 | to Spine1           |
| 199-bgw1    | 10.199.242.4/31  | Ethernet2       | 199-sp02      | Ethernet1/3 | to Spine2           |
| 199-bgw2    | 10.199.241.6/31  | Ethernet1       | 199-sp01      | Ethernet1/4 | to Spine1           |
| 199-bgw2    | 10.199.242.6/31  | Ethernet2       | 199-sp02      | Ethernet1/4 | to Spine2           |
| 199-sp01    | 10.199.241.1/31  | Ethernet1/1     | 199-lf01      | Ethernet1/1 | to lf01 Et1/1       |
| 199-sp01    | 10.199.241.3/31  | Ethernet1/2     | 199-lf02      | Ethernet1/1 | to lf02 Et1/1       |
| 199-sp01    | 10.199.241.5/31  | Ethernet1/3     | 199-bgw1      | Ethernet1   | to bgw1 Et1         |
| 199-sp01    | 10.199.241.7/31  | Ethernet1/4     | 199-bgw2      | Ethernet1   | to bgw2 Et1         |
| 199-sp02    | 10.199.242.1/31  | Ethernet1/1     | 199-lf01      | Ethernet1/2 | to lf01 Et1/2       |
| 199-sp02    | 10.199.242.3/31  | Ethernet1/2     | 199-lf02      | Ethernet1/2 | to lf02 Et1/2       |
| 199-sp02    | 10.199.242.5/31  | Ethernet1/3     | 199-bgw1      | Ethernet2   | to bgw1 Et2         |
| 199-sp02    | 10.199.242.7/31  | Ethernet1/4     | 199-bgw2      | Ethernet2   | to bgw2 Et2         |


### Loopback 0 адреса для BGP  POD 199 (Сеть 10.199.243.0/24)

| Device Name | Loopback0 Address | Description          |
|-------------|-------------------|----------------------|
| 199-lf1     | 10.199.243.1/32   | BGP Router-ID        |
| 199-lf2     | 10.199.243.2/32   | BGP Router-ID        |
| 199-bgw1    | 10.199.243.33/32  | BGP Router-ID        |
| 199-bgw2    | 10.199.243.44/32  | BGP Router-ID        |
| 199-sp1     | 10.199.243.11/32  | BGP Router-ID        |
| 199-sp2     | 10.199.243.22/32  | BGP Router-ID        |



### Loopback адреса для VXLAN NVE POD 199 (Сеть 10.199.244.0/24)

| Device Name | Loopback1 Address | Description          |
|-------------|-------------------|----------------------|
| 199-lf1     | 10.199.244.1/32   | VTEP Source          |
| 199-lf2     | 10.199.244.1/32   | VTEP Source          |
| 199-bgw1    | 10.199.244.33/32   | VTEP Source          |
| 199-bgw2    | 10.199.244.34/32   | VTEP Source          |

### MGMT адреса (Сеть 10.199.245.0/24)

| Device Name | Loopback0 Address | Description   |
|-------------|-------------------|---------------|
| 199-lf1     | 10.199.245.1/32   | Mgmt ip       |
| 199-lf2     | 10.199.245.2/32   | Mgmt ip       |
| 199-bgw2    | 10.199.245.3/32   | Mgmt ip       |
| 199-bgw2    | 10.199.245.4/32   | Mgmt ip       |
| 199-sp1     | 10.199.245.11/32  | Mgmt ip       |
| 199-sp2     | 10.199.245.22/32  | Mgmt ip       |



### EVPN L2-домены (VLAN ↔ VNI Mapping)

| VLAN | Описание          | L2 VNI | Anycast Gateway   | Leaf с настроенным VNI |
|------|-------------------|--------|-------------------|------------------------|
| 13   | VLAN13   | 19913  | 192.168.13.254/24  | 199-lf01, 199-lf02 VRF_CORE1 |
| 14   | VLAN14   | 19914  | 192.168.14.254/24  | 199-lf01, 199-lf02 VRF_CORE2 |
| 99   | VLAN99   | 19999  | -                  | 199-lf01, 199-lf02      |


### Конфигурация "ESXi" устройств 

| ESXi Device | Физические порты (Trunk) | Виртуальные интерфейсы (SVI)| IP Address/Маска  | Gateway      |
|-------------|--------------------------|-----------------------------|-------------------|--------------|
| 199-esx1    | Po1 Eth1→ 199-lf01 Eth1/5| Vlan13 (SVI для VLAN10)   | 192.168.13.1/24    | 192.168.13.254|
| 199-esx1    | Po1 Eth1→ 199-lf01 Eth1/5| Vlan13 (SVI для VLAN10)   | 192.168.13.1/24    | 192.168.13.254|
| 199-esx1    | Po1 Eth1→ 199-lf01 Eth1/5| Vlan14 (SVI для VLAN10)   | 192.168.14.2/24    | 192.168.14.254|
| 199-esx1    | Po1 Eth1→ 199-lf01 Eth1/5| Vlan14 (SVI для VLAN10)   | 192.168.14.2/24    | 192.168.14.254|
| 199-esx1    | Po1 Eth1→ 199-lf01 Eth1/5| Vlan99 (SVI для VLAN99)   | 192.168.99.1/24    | -             |
| 199-esx1    | Po1 Eth1→ 199-lf01 Eth1/5| Vlan99 (SVI для VLAN99)   | 192.168.99.1/24    | -             |
| 199-esx2    | Po1 Eth1→ 199-lf01 Eth1/6| Vlan13 (SVI для VLAN20)   | 192.168.13.1/24    | 192.168.13.254|
| 199-esx2    | Po1 Eth2→ 199-lf02 Eth1/6| Vlan13 (SVI для VLAN20)   | 192.168.13.1/24    | 192.168.13.254|
| 199-esx2    | Po1 Eth1→ 199-lf01 Eth1/6| Vlan14 (SVI для VLAN20)   | 192.168.14.1/24    | 192.168.14.254|
| 199-esx2    | Po1 Eth2→ 199-lf02 Eth1/6| Vlan14 (SVI для VLAN20)   | 192.168.14.1/24    | 192.168.14.254|
| 199-esx2    | Po1 Eth1→ 199-lf02 Eth1/6| Vlan99 (SVI для VLAN99)   | 192.168.99.2/24    | -             |
| 199-esx1    | Po1 Eth1→ 199-lf02 Eth1/6| Vlan99 (SVI для VLAN99)   | 192.168.99.2/24    | -             |

</details>




## DCI
<details>
<summary>IP Plan DCI </summary>
### Underlay сеть (DCI Links - Point-to-Point /31 из сети 10.88.241.0/24 10.88.242.0/24)

| Device Name | IP Address/Маска | Port            | Remote Device | Remote Port | Description         |
|-------------|------------------|-----------------|---------------|-------------|---------------------|
| 199-bgw1    | 10.88.241.0/31   | Ethernet3       | 99-blf01      | Ethernet6   | DCI 99-blf01        |
| 199-bgw2    | 10.88.242.0/31   | Ethernet3       | 99-blf02      | Ethernet6   | DCI 99-blf02        |

### Loopback 0 адреса для BGP  DCI (Сеть 10.88.243.0/24)

| Device Name | Loopback0 Address | Description          |
|-------------|-------------------|----------------------|
| 99-blf01    | 10.88.243.1/32    | BGP Router-ID        |
| 99-blf02    | 10.88.243.2/32    | BGP Router-ID        |
| 199-bgw1    | 10.88.243.33/32   | BGP Router-ID        |
| 199-bgw2    | 10.88.243.44/32   | BGP Router-ID        |

### Loopback адреса для VXLAN NVE DCI (Сеть 10.88.244.0/24)

| Device Name | Loopback1 Address | Description          |
|-------------|-------------------|----------------------|
| 99-blf01    | 10.88.244.1/32    | VTEP Source          |
| 99-blf02    | 10.88.244.2/32    | VTEP Source          |
| 99-blf01/02 | 10.88.244.12/32   | VTEP Source secondary|
| 199-bgw1    | 10.88.244.33/32   | VTEP Source          |
| 199-bgw2    | 10.88.244.34/32   | VTEP Source          |

</details>

-----------------------------------------------------------------------------------------------------------

## Конфигурация 

## POD-99
<details>
<summary>POD-199 </summary>

### 99-fw01 

<details>
<summary>FW01 </summary>

```bash
sysname 99-fw01
#
 undo l2tp sendaccm enable
 l2tp domain suffix-separator @
#
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
 firewall packet-filter basic-protocol enable
#
 firewall detect ftp
#
 firewall defend action discard
#
 log type traffic enable
 log type syslog enable
 log type policy enable
#
 undo dataflow enable
#
 isp name "china mobile"
 isp name "china mobile" set filename china-mobile.csv
 isp name "china unicom"
 isp name "china unicom" set filename china-unicom.csv
 isp name "china telecom"
 isp name "china telecom" set filename china-telecom.csv
 isp name "china educationnet"
 isp name "china educationnet" set filename china-educationnet.csv
#
 snmp-agent session history-max-number enable
 snmp-agent session trap threshold 1600000
 snmp-agent session-rate trap threshold 64000
#
 web-manager security version tlsv1 tlsv1.1
 web-manager security enable
#
firewall dataplane to manageplane application-apperceive default-action drop
#
 update schedule ips-sdb daily 04:26
 update schedule av-sdb daily 04:26
 update schedule sa-sdb daily 04:26
 update schedule cnc daily 04:26
#
ip vpn-instance default
 ipv4-family
#
 time-range worktime
  period-range 08:00:00 to 18:00:00 working-day
#
aaa
 authentication-scheme default
 authentication-scheme admin_local
 authentication-scheme admin_radius_local
 authentication-scheme admin_hwtacacs_local
 authentication-scheme admin_ad_local
 authentication-scheme admin_ldap_local
 authentication-scheme admin_radius
 authentication-scheme admin_hwtacacs
 authentication-scheme admin_ad
 authentication-scheme admin_ldap
 authorization-scheme default
 accounting-scheme default
 domain default
  service-type l2tp ike
  reference user current-domain
 manager-user password-modify enable
 manager-user audit-admin
  password cipher @%@%wFpR6#MRT1"egvP/@;R7[.2-$mp`2E[Z3&sM6LND_.:/.20[@%@%
  service-type web terminal
  level 15

 manager-user api-admin
  password cipher @%@%cnrC@:Yd#STb{d:Qm(_+]Q`4YkFgJCt)n&FtbJYE%1o;Q`7]@%@%
  service-type api
  level 15

 manager-user admin
  password cipher @%@%RqS9%jPme(=~B>N'q<F-QjAt,%-6KExumX]iVOO,O-}NjAwQ@%@%
  service-type ssh
  level 15

 role system-admin
  dashboard read-write
  monitor read-write
  policy read-write
  object read-write
  network read-write
  system read-write
 role device-admin
  dashboard read-only
  monitor read-only log log-traffic log-threat log-policy-matching report traffic-map threat-map session statistic statistic-acl
  monitor none diagnose
  policy read-write
  object read-write
  network read-write
  system read-write high-reliability
  system none configuration vsys license update-center mail-send feedback
 role device-admin(monitor)
  dashboard read-only
  monitor read-only log log-traffic log-threat log-policy-matching report traffic-map threat-map session statistic statistic-acl
  monitor none diagnose
  policy read-only
  object read-only
  network read-only
  system read-only high-reliability
  system none configuration vsys license update-center mail-send feedback
 role audit-admin
  dashboard read-only
  monitor read-write log-audit
  monitor read-only log log-traffic log-threat log-syslog log-policy-matching report traffic-map threat-map
  monitor none session statistic statistic-acl diagnose
  policy none
  object none
  network none
  system none
 bind manager-user audit-admin role audit-admin
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
interface GigabitEthernet1/0/0.14
 vlan-type dot1q 14
 description Gateway_for_VRF_14_VLAN14
 ip address 10.99.14.0 255.255.255.254
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
 add interface GigabitEthernet1/0/0.14
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
 peer blf01 timer keepalive 3 hold 9
 peer 10.99.1.1 as-number 65099
 peer 10.99.1.1 group blf01
 peer 10.99.2.1 as-number 65099
 peer 10.99.2.1 group blf01
 peer 10.99.3.1 as-number 65099
 peer 10.99.3.1 group blf01
 peer 10.99.4.1 as-number 65099
 peer 10.99.4.1 group blf01
 peer 10.99.14.1 as-number 65099
 peer 10.99.14.1 group blf01
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
 peer 10.99.14.2 as-number 65099
 peer 10.99.14.2 group blf02
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
  peer 10.99.14.1 enable
  peer 10.99.14.1 group blf01
  peer blf02 enable
  peer 10.99.1.2 enable
  peer 10.99.1.2 group blf02
  peer 10.99.2.2 enable
  peer 10.99.2.2 group blf02
  peer 10.99.3.2 enable
  peer 10.99.3.2 group blf02
  peer 10.99.4.2 enable
  peer 10.99.4.2 group blf02
  peer 10.99.14.2 enable
  peer 10.99.14.2 group blf02
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
#
sa
#
location
#
 multi-interface
  mode proportion-of-weight
#
right-manager server-group
#
api
#
security-policy
 rule name Underlay
  source-address 10.99.0.0 16
  source-address 10.99.241.0 24
  destination-address 10.99.0.0 16
  destination-address 10.99.241.0 24
  action permit
 rule name VRF_10_30
  source-zone dmz
  destination-zone trust
  source-address 192.168.3.0 24
  destination-address 192.168.1.1 32
  action permit
#
traffic-policy
#
policy-based-route
#
nat-policy
#
pcp-policy
#
dns-transparent-policy
#
return


 ```
 </details>

 ### 99-fw02 
<details>
<summary>FW02 </summary>

```bash
sysname 99-fw02
#
 undo l2tp sendaccm enable
 l2tp domain suffix-separator @
#
router id 10.99.245.253
#
undo telnet server enable
undo telnet ipv6 server enable
#
clock timezone UTC add 00:00:00
#
 hrp enable
 hrp standby-device
 hrp interface Eth-Trunk23 remote 10.99.241.12
 hrp standby config enable
 hrp adjust bgp-cost enable 20
 hrp track interface Eth-Trunk23
#
 firewall packet-filter basic-protocol enable
#
 firewall detect ftp
#
 firewall defend action discard
#
 log type traffic enable
 log type syslog enable
 log type policy enable
#
 undo dataflow enable
#
 isp name "china mobile"
 isp name "china mobile" set filename china-mobile.csv
 isp name "china unicom"
 isp name "china unicom" set filename china-unicom.csv
 isp name "china telecom"
 isp name "china telecom" set filename china-telecom.csv
 isp name "china educationnet"
 isp name "china educationnet" set filename china-educationnet.csv
#
 snmp-agent session history-max-number enable
 snmp-agent session trap threshold 1600000
 snmp-agent session-rate trap threshold 64000
#
 web-manager security version tlsv1 tlsv1.1
 web-manager security enable
#
firewall dataplane to manageplane application-apperceive default-action drop
#
 update schedule ips-sdb daily 00:15
 update schedule av-sdb daily 00:15
 update schedule sa-sdb daily 00:15
 update schedule cnc daily 00:15
#
ip vpn-instance default
 ipv4-family
#
 time-range worktime
  period-range 08:00:00 to 18:00:00 working-day
#
aaa
 authentication-scheme default
 authentication-scheme admin_local
 authentication-scheme admin_radius_local
 authentication-scheme admin_hwtacacs_local
 authentication-scheme admin_ad_local
 authentication-scheme admin_ldap_local
 authentication-scheme admin_radius
 authentication-scheme admin_hwtacacs
 authentication-scheme admin_ad
 authentication-scheme admin_ldap
 authorization-scheme default
 accounting-scheme default
 domain default
  service-type l2tp ike
  reference user current-domain
 manager-user password-modify enable
 manager-user audit-admin
  password cipher @%@%S;eN$Qpa>KYT7wGUp)v=Q#Pf2gKz'W2[8U_XmaTR/(K.#PiQ@%@%
  service-type web terminal
  level 15

 manager-user api-admin
  password cipher @%@%`-QYNJEv[Dvnwk4Pq>q(SPqi^9<BUWY8[FB'H]G1%6/VPqlS@%@%
  service-type api
  level 15

 manager-user admin
  password cipher @%@%6X1EXVD1TU(N)$B,}@y<o1Oft4EVWeI1X-mo(x>:8R2B1Oio@%@%
  service-type ssh
  level 15

 role system-admin
  dashboard read-write
  monitor read-write
  policy read-write
  object read-write
  network read-write
  system read-write
 role device-admin
  dashboard read-only
  monitor read-only log log-traffic log-threat log-policy-matching report traffic-map threat-map session statistic statistic-acl
  monitor none diagnose
  policy read-write
  object read-write
  network read-write
  system read-write high-reliability
  system none configuration vsys license update-center mail-send feedback
 role device-admin(monitor)
  dashboard read-only
  monitor read-only log log-traffic log-threat log-policy-matching report traffic-map threat-map session statistic statistic-acl
  monitor none diagnose
  policy read-only
  object read-only
  network read-only
  system read-only high-reliability
  system none configuration vsys license update-center mail-send feedback
 role audit-admin
  dashboard read-only
  monitor read-write log-audit
  monitor read-only log log-traffic log-threat log-syslog log-policy-matching report traffic-map threat-map
  monitor none session statistic statistic-acl diagnose
  policy none
  object none
  network none
  system none
 bind manager-user audit-admin role audit-admin
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
interface GigabitEthernet1/0/0.14
 vlan-type dot1q 14
 description Gateway_for_VRF_14_VLAN14
 ip address 10.99.14.3 255.255.255.254
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
 add interface GigabitEthernet1/0/0.14
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
 peer 10.99.14.1 as-number 65099
 peer 10.99.14.1 group blf01
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
 peer 10.99.14.2 as-number 65099
 peer 10.99.14.2 group blf02
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
  peer 10.99.14.1 enable
  peer 10.99.14.1 group blf01
  peer blf02 enable
  peer 10.99.1.2 enable
  peer 10.99.1.2 group blf02
  peer 10.99.2.2 enable
  peer 10.99.2.2 group blf02
  peer 10.99.3.2 enable
  peer 10.99.3.2 group blf02
  peer 10.99.4.2 enable
  peer 10.99.4.2 group blf02
  peer 10.99.14.2 enable
  peer 10.99.14.2 group blf02
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
#
sa
#
location
#
 multi-interface
  mode proportion-of-weight
#
right-manager server-group
#
api
#
security-policy
 rule name Underlay
  source-address 10.99.0.0 16
  source-address 10.99.241.0 24
  source-address 10.99.242.0 24
  destination-address 10.99.0.0 16
  destination-address 10.99.241.0 24
  destination-address 10.99.242.0 24
  action permit
 rule name VRF_10_30
  source-zone dmz
  destination-zone trust
  source-address 192.168.3.0 24
  destination-address 192.168.1.1 32
  action permit
#
traffic-policy
#
policy-based-route
#
nat-policy
#
pcp-policy
#
dns-transparent-policy
#
return


 ```
</details>

### 99-blf1 (Border Leaf 1)

<details>
<summary>99-blf1 </summary>

```bash
hostname 99-blf1
!
spanning-tree mode mstp
spanning-tree edge-port bpduguard default
!
vlan 10
   name VLAN10
!
vlan 20
   name VLAN20
!
vlan 99
!
vlan 4094
   name MLAG-PEERLINK
   trunk group MLAG-PEERLINK
!
vrf instance MGMT
!
vrf instance VRF_13
!
vrf instance VRF_14
!
vrf instance VRF_CORE_1
!
vrf instance VRF_CORE_2
!
vrf instance VRF_CORE_3
!
vrf instance VRF_CORE_4
!
interface Port-Channel3
   description 99-esx1
   mtu 9100
   switchport trunk allowed vlan 10,20,99
   switchport mode trunk
   mlag 3
!
interface Port-Channel5
   mtu 9100
   switchport trunk allowed vlan 10,20
   switchport mode trunk
   mlag 5
!
interface Port-Channel78
   description MLAG_PEERLINK
   switchport mode trunk
   switchport trunk group MLAG-PEERLINK
   spanning-tree link-type point-to-point
!
interface Ethernet1
   description to-99-sp1-Eth1
   mtu 9100
   no switchport
   ip address 10.99.241.0/31
!
interface Ethernet2
   description to-99-sp2-Eth1
   mtu 9100
   no switchport
   ip address 10.99.242.0/31
!
interface Ethernet3
   description to-99-esx1-Eth1
   mtu 9100
   switchport trunk allowed vlan 10,20,90
   switchport mode trunk
   channel-group 3 mode active
!
interface Ethernet4
   description 99-fw01 G1/0/0
   no switchport
!
interface Ethernet4.10
   encapsulation dot1q vlan 10
   vrf VRF_CORE_1
   ip address 10.99.1.1/31
!
interface Ethernet4.14
   encapsulation dot1q vlan 14
   vrf VRF_14
   ip address 10.99.14.1/31
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
!
interface Ethernet5
   description 99-esx2-eth2
   mtu 9100
   speed forced 10000full
   switchport trunk allowed vlan 10,20
   switchport mode trunk
   channel-group 5 mode active
!
interface Ethernet6
   description to-199-bgw1-Eth3
   mtu 9100
   no switchport
   ip address 10.88.241.1/31
!
interface Ethernet7
   description Po78 lf2
   channel-group 78 mode active
!
interface Ethernet8
   description Po78 lf2
   channel-group 78 mode active
!
interface Ethernet9
!
interface Ethernet10
!
interface Loopback0
   description Router-ID
   ip address 10.99.243.1/32
!
interface Loopback1
   description VXLAN-Tunnel-Endpoint
   ip address 10.99.244.1/32
   ip address 10.99.244.12/32 secondary
!
interface Loopback10
   description BGP-Router-ID-DCI
   ip address 10.88.243.1/32
!
interface Loopback11
   description VTEP-Source-DCI
   ip address 10.88.244.1/32
!
interface Loopback13
   vrf VRF_13
   ip address 192.168.233.11/24
!
interface Loopback14
   vrf VRF_14
   ip address 192.168.244.11/24
!
interface Management1
   description MGMT
   vrf MGMT
   ip address 10.99.245.1/24
!
interface Vlan10
   description Vlan10
   mtu 9100
   vrf VRF_CORE_1
   ip address virtual 192.168.1.254/24
!
interface Vlan20
   description Vlan20
   mtu 9100
   vrf VRF_CORE_2
   ip address virtual 192.168.2.254/24
!
interface Vlan4094
   no autostate
   ip address 10.99.246.1/30
!
interface Vxlan1
   description VXLAN-Tunnel-Endpoint
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
   vxlan vlan 99 vni 19999
   vxlan vrf VRF_13 vni 19913
   vxlan vrf VRF_14 vni 19914
   vxlan vrf VRF_CORE_1 vni 10001
   vxlan vrf VRF_CORE_2 vni 10002
   vxlan vrf VRF_CORE_3 vni 10003
   vxlan vrf VRF_CORE_4 vni 10004
   vxlan learn-restrict any
!
ip virtual-router mac-address 00:00:00:00:00:01
!
ip routing
ip routing vrf MGMT
ip routing vrf VRF_13
ip routing vrf VRF_14
ip routing vrf VRF_CORE_1
ip routing vrf VRF_CORE_2
ip routing vrf VRF_CORE_3
ip routing vrf VRF_CORE_4
!
ip prefix-list DEFAULT
   seq 10 permit 0.0.0.0/0
!
ip prefix-list DEFAULT_3
   seq 10 permit 0.0.0.0/0
   seq 20 permit 192.168.3.0/24 le 32
   seq 30 permit 172.16.30.0/24 le 32
!
ip prefix-list DEFAULT_4
   seq 10 permit 0.0.0.0/0
   seq 20 permit 192.168.4.0/24 le 32
   seq 30 deny 192.168.0.0/16 le 32
!
ip prefix-list PL_BLF_TO_DCI
   seq 5 permit 10.88.243.1/32
   seq 10 permit 10.88.244.1/32
!
ip prefix-list PL_BLF_TO_SPINE
   seq 5 permit 10.99.243.1/32
   seq 10 permit 10.99.244.1/32
   seq 15 permit 10.88.243.1/32
   seq 20 permit 10.88.244.1/32
!
ip prefix-list VRF_CORES
   seq 10 permit 192.168.0.0/16 le 32
!
ip prefix-list VRF_CORE_4
   seq 10 permit 192.168.4.0/24
!
mlag configuration
   domain-id 12
   local-interface Vlan4094
   peer-address 10.99.246.2
   peer-address heartbeat 10.99.245.2 vrf MGMT
   peer-link Port-Channel78
   dual-primary detection delay 1 action errdisable all-interfaces
!
ip route vrf MGMT 0.0.0.0/0 10.99.245.254
!
route-map DEFAULT permit 10
   match ip address prefix-list DEFAULT
!
route-map RM_BLF_TO_DCI permit 10
   match ip address prefix-list PL_BLF_TO_DCI
!
route-map RM_BLF_TO_SPINE permit 10
   match ip address prefix-list PL_BLF_TO_SPINE
!
route-map VRFS_CORE permit 10
   match ip address prefix-list VRF_CORES
!
route-map VRF_CORE_3 permit 10
   match ip address prefix-list DEFAULT_3
!
route-map VRF_CORE_4 permit 10
   match ip address prefix-list DEFAULT_4
!
router bgp 65099
   router-id 10.99.243.1
   no bgp default ipv4-unicast
   timers bgp 3 9
   rd auto
   maximum-paths 10 ecmp 10
   neighbor DCI-EVPN peer group
   neighbor DCI-EVPN remote-as 65999
   neighbor DCI-EVPN update-source Loopback10
   neighbor DCI-EVPN ebgp-multihop 20
   neighbor DCI-EVPN send-community extended
   neighbor DCI-UNDERLAY peer group
   neighbor DCI-UNDERLAY remote-as 65999
   neighbor DCI-UNDERLAY ebgp-multihop 10
   neighbor DCI-UNDERLAY timers 3 9
   neighbor DCI-UNDERLAY send-community
   neighbor SPINE-EVPN peer group
   neighbor SPINE-EVPN remote-as 65099
   neighbor SPINE-EVPN update-source Loopback0
   neighbor SPINE-EVPN ebgp-multihop 2
   neighbor SPINE-EVPN send-community extended
   neighbor SPINE-UNDERLAY peer group
   neighbor SPINE-UNDERLAY remote-as 65099
   neighbor SPINE-UNDERLAY timers 3 9
   neighbor SPINE-UNDERLAY send-community
   neighbor 10.88.241.0 peer group DCI-UNDERLAY
   neighbor 10.88.243.33 peer group DCI-EVPN
   neighbor 10.99.1.3 ebgp-multihop 10
   neighbor 10.99.241.1 peer group SPINE-UNDERLAY
   neighbor 10.99.242.1 peer group SPINE-UNDERLAY
   neighbor 10.99.243.11 peer group SPINE-EVPN
   neighbor 10.99.243.22 peer group SPINE-EVPN
   neighbor 10.99.246.2 remote-as 65099
   neighbor 10.99.246.2 next-hop-self
   neighbor 10.99.246.2 send-community
   !
   vlan 10
      rd auto
      route-target both 65099:10010
      redistribute learned
   !
   vlan 99
      rd evpn domain all 10.88.243.1:99
      route-target both 65099:19999
      route-target import evpn domain remote 65999:19999
      route-target export evpn domain remote 65999:19999
      redistribute learned
   !
   address-family evpn
      neighbor DCI-EVPN activate
      neighbor DCI-EVPN default-route
      neighbor DCI-EVPN domain remote
      neighbor SPINE-EVPN activate
      domain identifier 2:2
      neighbor default next-hop-self received-evpn-routes route-type ip-prefix inter-domain
   !
   address-family ipv4
      neighbor DCI-UNDERLAY activate
      no neighbor SPINE-EVPN activate
      neighbor SPINE-UNDERLAY activate
      neighbor SPINE-UNDERLAY route-map RM_BLF_TO_SPINE out
      neighbor 10.99.246.2 activate
      network 10.88.243.1/32
      network 10.88.244.1/32
      network 10.99.243.1/32
      network 10.99.244.1/32
      network 10.99.244.12/32
   !
   vrf VRF_13
      rd 65099:19913
      route-target import evpn 65199:19913
      route-target export evpn 65199:19913
      !
      address-family ipv4
         redistribute connected
   !
   vrf VRF_14
      rd 65099:19914
      route-target import evpn 65199:19914
      route-target export evpn 65199:19914
      router-id 10.99.14.1
      timers bgp 60 180 min-hold-time 3
      neighbor 10.99.14.0 remote-as 65098
      neighbor 10.99.14.0 next-hop-self
      neighbor 10.99.14.0 update-source Ethernet4.14
      neighbor 10.99.14.0 route-map DEFAULT in
      neighbor 10.99.14.3 remote-as 65098
      neighbor 10.99.14.3 next-hop-self
      neighbor 10.99.14.3 update-source Ethernet4.14
      neighbor 10.99.14.3 ebgp-multihop 10
      neighbor 10.99.14.3 route-map DEFAULT in
      !
      address-family ipv4
         no neighbor 10.99.2.0 activate
         neighbor 10.99.14.0 activate
         neighbor 10.99.14.3 activate
         network 10.99.14.0/31
         redistribute connected
   !
   vrf VRF_CORE_1
      rd 65099:1011
      route-target import evpn 65099:101
      route-target export evpn 65099:101
      router-id 10.99.1.1
      timers bgp 60 180 min-hold-time 3
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
   !
   vrf VRF_CORE_2
      rd 65099:1021
      route-target import evpn 65099:102
      route-target export evpn 65099:102
      router-id 10.99.2.1
      timers bgp 60 180 min-hold-time 3
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
   !
   vrf VRF_CORE_3
      rd 65099:1031
      route-target import evpn 65099:103
      route-target export evpn 65099:103
      router-id 10.99.3.1
      timers bgp 60 180 min-hold-time 3
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
   !
   vrf VRF_CORE_4
      rd 65099:1041
      route-target import evpn 65099:104
      route-target export evpn 65099:104
      router-id 10.99.4.1
      timers bgp 60 180 min-hold-time 3
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
!
end




 ```
 </details>

 
 ### 99-blf2 (Border Leaf 2)

<details>
<summary>99-Blf2 </summary>

 ```bash
hostname 99-blf2
!
spanning-tree mode mstp
spanning-tree edge-port bpduguard default
!
vlan 10
   name VLAN10
!
vlan 20
   name VLAN20
!
vlan 99
!
vlan 4094
   name MLAG-PEERLINK
   trunk group MLAG-PEERLINK
!
vrf instance MGMT
!
vrf instance VRF_13
!
vrf instance VRF_14
!
vrf instance VRF_CORE_1
!
vrf instance VRF_CORE_2
!
vrf instance VRF_CORE_3
!
vrf instance VRF_CORE_4
!
interface Port-Channel3
   description 99-esx1
   mtu 9100
   switchport trunk allowed vlan 10,20,99
   switchport mode trunk
   mlag 3
!
interface Port-Channel5
   mtu 9100
   switchport trunk allowed vlan 10,20
   switchport mode trunk
   mlag 5
!
interface Port-Channel78
   description MLAG_PEERLINK
   switchport mode trunk
   switchport trunk group MLAG-PEERLINK
   spanning-tree link-type point-to-point
!
interface Ethernet1
   description to-99-sp1-Eth2
   mtu 9100
   no switchport
   ip address 10.99.241.2/31
!
interface Ethernet2
   description to-99-sp2-Eth2
   mtu 9100
   no switchport
   ip address 10.99.242.2/31
!
interface Ethernet3
   description 99-esx1 et2
   mtu 9100
   switchport trunk allowed vlan 10,20
   switchport mode trunk
   channel-group 3 mode active
!
interface Ethernet4
   description 99-fw01 G1/0/0
   mtu 9100
   no switchport
!
interface Ethernet4.10
   encapsulation dot1q vlan 10
   vrf VRF_CORE_1
   ip address 10.99.1.2/31
!
interface Ethernet4.14
   encapsulation dot1q vlan 14
   vrf VRF_14
   ip address 10.99.14.2/31
!
interface Ethernet4.20
   encapsulation dot1q vlan 20
   vrf VRF_CORE_2
   ip address 10.99.2.2/31
!
interface Ethernet4.30
   encapsulation dot1q vlan 30
   vrf VRF_CORE_3
   ip address 10.99.3.2/31
!
interface Ethernet4.40
   encapsulation dot1q vlan 40
   vrf VRF_CORE_4
   ip address 10.99.4.2/31
!
interface Ethernet5
   description 99-esx2 et1
   mtu 9100
   speed forced 10000full
   switchport trunk allowed vlan 10,20
   switchport mode trunk
   channel-group 5 mode active
!
interface Ethernet6
   description to-199-bgw2-Eth3
   mtu 9100
   no switchport
   ip address 10.88.242.1/31
!
interface Ethernet7
   description Po78 lf1
   channel-group 78 mode active
!
interface Ethernet8
   description Po78 lf1
   channel-group 78 mode active
!
interface Ethernet9
!
interface Ethernet10
!
interface Loopback0
   description Router-ID
   ip address 10.99.243.2/32
!
interface Loopback1
   description VTEP-Source
   ip address 10.99.244.2/32
   ip address 10.99.244.12/32 secondary
!
interface Loopback10
   description BGP-Router-ID-DCI
   ip address 10.88.243.2/32
!
interface Loopback11
   description VTEP-Source-DCI
   ip address 10.88.244.2/32
!
interface Management1
   description MGMT
   vrf MGMT
   ip address 10.99.245.2/24
!
interface Vlan10
   description Vlan10
   mtu 9100
   vrf VRF_CORE_1
   ip address virtual 192.168.1.100/24
!
interface Vlan20
   description Vlan20
   mtu 9100
   vrf VRF_CORE_2
   ip address virtual 192.168.2.100/24
!
interface Vlan4094
   no autostate
   ip address 10.99.246.2/30
!
interface Vxlan1
   description VXLAN-Tunnel-Endpoint
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
   vxlan vlan 99 vni 19999
   vxlan vrf VRF_13 vni 19913
   vxlan vrf VRF_14 vni 19914
   vxlan vrf VRF_CORE_1 vni 10001
   vxlan vrf VRF_CORE_2 vni 10002
   vxlan vrf VRF_CORE_3 vni 10003
   vxlan vrf VRF_CORE_4 vni 10004
   vxlan learn-restrict any
!
ip virtual-router mac-address 00:00:00:00:00:01
!
ip routing
ip routing vrf MGMT
ip routing vrf VRF_13
ip routing vrf VRF_14
ip routing vrf VRF_CORE_1
ip routing vrf VRF_CORE_2
ip routing vrf VRF_CORE_3
ip routing vrf VRF_CORE_4
!
ip prefix-list DEFAULT
   seq 10 permit 0.0.0.0/0
!
ip prefix-list PL_BLF_TO_DCI
   seq 5 permit 10.88.243.2/32
   seq 10 permit 10.88.244.2/32
!
ip prefix-list PL_BLF_TO_SPINE
   seq 5 permit 10.99.243.2/32
   seq 10 permit 10.99.244.2/32
   seq 15 permit 10.88.243.2/32
   seq 20 permit 10.88.244.2/32
!
mlag configuration
   domain-id 12
   local-interface Vlan4094
   peer-address 10.99.246.1
   peer-address heartbeat 10.99.245.1 vrf MGMT
   peer-link Port-Channel78
   dual-primary detection delay 1 action errdisable all-interfaces
!
ip route vrf MGMT 0.0.0.0/0 10.99.245.254
!
route-map DEFAULT permit 10
   match ip address prefix-list DEFAULT
!
route-map RM_BLF_TO_DCI permit 10
   match ip address prefix-list PL_BLF_TO_DCI
!
route-map RM_BLF_TO_SPINE permit 10
   match ip address prefix-list PL_BLF_TO_SPINE
!
router bgp 65099
   router-id 10.99.243.2
   no bgp default ipv4-unicast
   timers bgp 3 9
   maximum-paths 10 ecmp 10
   neighbor DCI-EVPN peer group
   neighbor DCI-EVPN remote-as 65999
   neighbor DCI-EVPN update-source Loopback10
   neighbor DCI-EVPN ebgp-multihop 20
   neighbor DCI-EVPN send-community extended
   neighbor DCI-UNDERLAY peer group
   neighbor DCI-UNDERLAY remote-as 65999
   neighbor DCI-UNDERLAY ebgp-multihop 10
   neighbor DCI-UNDERLAY timers 3 9
   neighbor DCI-UNDERLAY send-community
   neighbor SPINE-EVPN peer group
   neighbor SPINE-EVPN remote-as 65099
   neighbor SPINE-EVPN update-source Loopback0
   neighbor SPINE-EVPN ebgp-multihop 2
   neighbor SPINE-EVPN send-community extended
   neighbor SPINE-UNDERLAY peer group
   neighbor SPINE-UNDERLAY remote-as 65099
   neighbor SPINE-UNDERLAY timers 3 9
   neighbor SPINE-UNDERLAY send-community
   neighbor 10.88.242.0 peer group DCI-UNDERLAY
   neighbor 10.88.243.44 peer group DCI-EVPN
   neighbor 10.99.241.3 peer group SPINE-UNDERLAY
   neighbor 10.99.242.3 peer group SPINE-UNDERLAY
   neighbor 10.99.243.11 peer group SPINE-EVPN
   neighbor 10.99.243.22 peer group SPINE-EVPN
   neighbor 10.99.246.1 remote-as 65099
   neighbor 10.99.246.1 next-hop-self
   neighbor 10.99.246.1 send-community
   !
   vlan 10
      rd auto
      route-target both 65099:10010
      redistribute learned
   !
   vlan 99
      rd auto
      route-target both 65099:19999
      route-target import export evpn domain remote 65199:19999
      route-target import export evpn domain remote 65999:19999
      redistribute learned
   !
   address-family evpn
      neighbor DCI-EVPN activate
      neighbor DCI-EVPN default-route
      neighbor DCI-EVPN domain remote
      neighbor SPINE-EVPN activate
      domain identifier 2:2
      neighbor default next-hop-self received-evpn-routes route-type ip-prefix inter-domain
   !
   address-family ipv4
      neighbor DCI-UNDERLAY activate
      neighbor SPINE-UNDERLAY activate
      neighbor SPINE-UNDERLAY route-map RM_BLF_TO_SPINE out
      no neighbor 10.99.4.0 activate
      no neighbor 10.99.4.3 activate
      no neighbor 10.99.14.0 activate
      no neighbor 10.99.14.3 activate
      neighbor 10.99.246.1 activate
      network 10.88.243.2/32
      network 10.88.244.2/32
      network 10.99.243.2/32
      network 10.99.244.2/32
      network 10.99.244.12/32
   !
   vrf VRF_13
      rd 65099:19913
      route-target import evpn 65199:19913
      route-target export evpn 65199:19913
      !
      address-family ipv4
         redistribute connected
   !
   vrf VRF_14
      rd 65092:19914
      route-target import evpn 65199:19914
      route-target export evpn 65199:19914
      router-id 10.99.14.2
      timers bgp 60 180 min-hold-time 3
      neighbor 10.99.14.0 remote-as 65098
      neighbor 10.99.14.0 next-hop-self
      neighbor 10.99.14.0 update-source Ethernet4.14
      neighbor 10.99.14.0 ebgp-multihop 10
      neighbor 10.99.14.0 route-map DEFAULT in
      neighbor 10.99.14.3 remote-as 65098
      neighbor 10.99.14.3 next-hop-self
      neighbor 10.99.14.3 update-source Ethernet4.14
      neighbor 10.99.14.3 ebgp-multihop 10
      neighbor 10.99.14.3 route-map DEFAULT in
      !
      address-family ipv4
         no neighbor 10.99.4.0 activate
         no neighbor 10.99.4.3 activate
         neighbor 10.99.14.0 activate
         neighbor 10.99.14.3 activate
         network 10.99.14.2/31
         redistribute connected
   !
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
   !
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
   !
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
   !
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
!
end



 ```
</details>

### 99-lf3 (Leaf 3)

<details>
<summary>99-Lf3 </summary>

```bash
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname 99-lf3
!
spanning-tree mode mstp
!
vlan 10
   name VLAN10
!
vlan 20
   name VLAN20
!
vlan 30
   name VLAN30
!
vlan 40
   name VLAN40
!
vlan 99
!
vrf instance MGMT
!
vrf instance VRF_13
!
vrf instance VRF_CORE_1
!
vrf instance VRF_CORE_2
!
vrf instance VRF_CORE_3
!
vrf instance VRF_CORE_4
!
interface Port-Channel3
   switchport trunk allowed vlan 30
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0000:0000:0000:0000:3403
      designated-forwarder election algorithm preference 100
      route-target import 00:00:00:00:34:03
   lacp system-id 0000.0000.3403
!
interface Port-Channel4
   switchport trunk allowed vlan 40,99
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0000:0000:0000:0000:3404
      designated-forwarder election algorithm preference 100
      route-target import 00:00:00:00:34:04
   lacp system-id 0000.0000.3404
!
interface Ethernet1
   description to-99-sp1-Eth3
   mtu 9100
   no switchport
   ip address 10.99.241.4/31
!
interface Ethernet2
   description to-99-sp2-Eth3
   mtu 9100
   no switchport
   ip address 10.99.242.4/31
!
interface Ethernet3
   description to esx3 int et 1
   mtu 9100
   switchport trunk allowed vlan 30
   switchport mode trunk
   channel-group 3 mode active
!
interface Ethernet4
   description to esx4 et1
   mtu 9100
   switchport trunk allowed vlan 40
   switchport mode trunk
   channel-group 4 mode active
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Loopback0
   description Router-ID
   ip address 10.99.243.3/32
!
interface Loopback1
   description VTEP-Source
   ip address 10.99.244.3/32
!
interface Loopback13
   vrf VRF_13
   ip address 192.168.134.3/32
!
interface Management1
   vrf MGMT
   ip address 10.99.245.3/24
!
interface Vlan30
   description Vlan30
   mtu 9100
   vrf VRF_CORE_3
   ip address virtual 192.168.3.254/24
!
interface Vlan40
   description Vlan40
   mtu 9100
   vrf VRF_CORE_4
   ip address virtual 192.168.4.254/24
!
interface Vxlan1
   description VXLAN-Tunnel-Endpoint
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 30 vni 10030
   vxlan vlan 40 vni 10040
   vxlan vlan 99 vni 19999
   vxlan vrf VRF_13 vni 19913
   vxlan vrf VRF_CORE_1 vni 10001
   vxlan vrf VRF_CORE_2 vni 10002
   vxlan vrf VRF_CORE_3 vni 10003
   vxlan vrf VRF_CORE_4 vni 10004
   vxlan learn-restrict any
!
ip virtual-router mac-address 00:00:00:00:00:01
!
ip routing
ip routing vrf MGMT
ip routing vrf VRF_13
ip routing vrf VRF_CORE_1
ip routing vrf VRF_CORE_2
ip routing vrf VRF_CORE_3
ip routing vrf VRF_CORE_4
!
ip route vrf MGMT 0.0.0.0/0 10.99.245.254
!
router bgp 65099
   router-id 10.99.243.3
   no bgp default ipv4-unicast
   timers bgp 3 9
   maximum-paths 10 ecmp 10
   neighbor SPINE-EVPN peer group
   neighbor SPINE-EVPN remote-as 65099
   neighbor SPINE-EVPN update-source Loopback0
   neighbor SPINE-EVPN ebgp-multihop 2
   neighbor SPINE-EVPN send-community extended
   neighbor SPINE-UNDERLAY peer group
   neighbor SPINE-UNDERLAY remote-as 65099
   neighbor SPINE-UNDERLAY timers 3 9
   neighbor SPINE-UNDERLAY send-community
   neighbor 10.99.241.5 peer group SPINE-UNDERLAY
   neighbor 10.99.242.5 peer group SPINE-UNDERLAY
   neighbor 10.99.243.11 peer group SPINE-EVPN
   neighbor 10.99.243.22 peer group SPINE-EVPN
   !
   vlan 30
      rd auto
      route-target both 65099:10030
      redistribute learned
   !
   vlan 40
      rd auto
      route-target both 65099:10040
      redistribute learned
   !
   vlan 99
      rd auto
      route-target both 65099:19999
      redistribute learned
   !
   address-family evpn
      neighbor SPINE-EVPN activate
   !
   address-family ipv4
      neighbor SPINE-UNDERLAY activate
      network 10.99.243.3/32
      network 10.99.244.3/32
   !
   vrf VRF_13
      rd 65881:19913
      route-target import evpn 65099:103
      route-target import evpn 65199:19913
      route-target export evpn 65099:103
      route-target export evpn 65199:19913
      !
      address-family ipv4
         redistribute connected
   !
   vrf VRF_CORE_1
      rd 65099:1013
      route-target import evpn 65099:101
      route-target export evpn 65099:101
      !
      address-family ipv4
         redistribute connected
   !
   vrf VRF_CORE_2
      rd 65099:1023
      route-target import evpn 65099:102
      route-target export evpn 65099:102
      !
      address-family ipv4
         redistribute connected
   !
   vrf VRF_CORE_3
      rd 65099:1033
      route-target import evpn 65099:103
      route-target export evpn 65099:103
      !
      address-family ipv4
         redistribute connected
   !
   vrf VRF_CORE_4
      rd 65099:1043
      route-target import evpn 65099:104
      route-target export evpn 65099:104
      !
      address-family ipv4
         redistribute connected
!
end


 ```
</details>

 ### 99-lf4 (Leaf 4)

 <details>
<summary>99-Lf4 </summary>

```bash
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname 99-lf4
!
spanning-tree mode mstp
!
vlan 10
   name VLAN10
!
vlan 20
   name VLAN20
!
vlan 30
   name VLAN30
!
vlan 40
   name vlan40
!
vlan 99
!
vrf instance MGMT
!
vrf instance VRF_13
!
vrf instance VRF_CORE_1
!
vrf instance VRF_CORE_2
!
vrf instance VRF_CORE_3
!
vrf instance VRF_CORE_4
!
interface Port-Channel3
   switchport trunk allowed vlan 30
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0000:0000:0000:0000:3403
      designated-forwarder election algorithm preference 90
      route-target import 00:00:00:00:34:03
   lacp system-id 0000.0000.3403
!
interface Port-Channel4
   switchport trunk allowed vlan 40,99
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0000:0000:0000:0000:3404
      designated-forwarder election algorithm preference 100
      route-target import 00:00:00:00:34:04
   lacp system-id 0000.0000.3404
!
interface Ethernet1
   description to-99-sp1-Eth4
   mtu 9100
   no switchport
   ip address 10.99.241.6/31
!
interface Ethernet2
   description to-99-sp2-Eth4
   mtu 9100
   no switchport
   ip address 10.99.242.6/31
!
interface Ethernet3
   description to esx3 int et2
   mtu 9100
   switchport trunk allowed vlan 30
   switchport mode trunk
   channel-group 3 mode active
!
interface Ethernet4
   description to esx4 int et 2
   mtu 9100
   switchport trunk allowed vlan 40
   switchport mode trunk
   channel-group 4 mode active
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Loopback0
   description Router-ID
   ip address 10.99.243.4/32
!
interface Loopback1
   description VTEP-Source
   ip address 10.99.244.4/32
!
interface Loopback13
   vrf VRF_13
   ip address 192.168.134.4/32
!
interface Management1
   vrf MGMT
   ip address 10.99.245.4/24
!
interface Vlan30
   description Vlan30
   mtu 9100
   vrf VRF_CORE_3
   ip address virtual 192.168.3.254/24
!
interface Vlan40
   description Vlan40
   mtu 9100
   vrf VRF_CORE_4
   ip address virtual 192.168.4.254/24
!
interface Vxlan1
   description VXLAN-Tunnel-Endpoint
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 30 vni 10030
   vxlan vlan 40 vni 10040
   vxlan vlan 99 vni 19999
   vxlan vrf VRF_13 vni 19913
   vxlan vrf VRF_CORE_1 vni 10001
   vxlan vrf VRF_CORE_2 vni 10002
   vxlan vrf VRF_CORE_3 vni 10003
   vxlan vrf VRF_CORE_4 vni 10004
   vxlan learn-restrict any
!
ip virtual-router mac-address 00:00:00:00:00:01
!
ip routing
ip routing vrf MGMT
ip routing vrf VRF_13
ip routing vrf VRF_CORE_1
ip routing vrf VRF_CORE_2
ip routing vrf VRF_CORE_3
ip routing vrf VRF_CORE_4
!
ip route vrf MGMT 0.0.0.0/0 10.99.245.254
!
router bgp 65099
   router-id 10.99.243.4
   no bgp default ipv4-unicast
   timers bgp 3 9
   maximum-paths 10 ecmp 10
   neighbor SPINE-EVPN peer group
   neighbor SPINE-EVPN remote-as 65099
   neighbor SPINE-EVPN update-source Loopback0
   neighbor SPINE-EVPN ebgp-multihop 2
   neighbor SPINE-EVPN send-community extended
   neighbor SPINE-UNDERLAY peer group
   neighbor SPINE-UNDERLAY remote-as 65099
   neighbor SPINE-UNDERLAY timers 3 9
   neighbor SPINE-UNDERLAY send-community
   neighbor 10.99.241.7 peer group SPINE-UNDERLAY
   neighbor 10.99.242.7 peer group SPINE-UNDERLAY
   neighbor 10.99.243.11 peer group SPINE-EVPN
   neighbor 10.99.243.22 peer group SPINE-EVPN
   !
   vlan 30
      rd auto
      route-target both 65099:10030
      redistribute learned
   !
   vlan 40
      rd auto
      route-target both 65099:10040
      redistribute learned
   !
   vlan 99
      rd auto
      route-target both 65099:19999
      redistribute learned
   !
   address-family evpn
      neighbor SPINE-EVPN activate
   !
   address-family ipv4
      neighbor SPINE-UNDERLAY activate
      network 10.99.243.4/32
      network 10.99.244.4/32
   !
   vrf VRF_13
      rd 65099:19913
      route-target import evpn 65199:19913
      route-target export evpn 65199:19913
      !
      address-family ipv4
         redistribute connected
   !
   vrf VRF_CORE_1
      rd 65099:1014
      route-target import evpn 65099:101
      route-target export evpn 65099:101
      !
      address-family ipv4
         redistribute connected
   !
   vrf VRF_CORE_2
      rd 65099:1024
      route-target import evpn 65099:102
      route-target export evpn 65099:102
      !
      address-family ipv4
         redistribute connected
   !
   vrf VRF_CORE_3
      rd 65099:1034
      route-target import evpn 65099:103
      route-target export evpn 65099:103
      !
      address-family ipv4
         redistribute connected
   !
   vrf VRF_CORE_4
      rd 65099:1044
      route-target import evpn 65099:104
      route-target export evpn 65099:104
      !
      address-family ipv4
         redistribute connected
!
end


 ```
</details>

 ### 99-sp1 (Spine 1)
<details>
<summary>99-Sp1 </summary>


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
</details>


 ### 99-sp2 (Spine 2)

<details>
<summary>99-Sp2 </summary>

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
 </details>
 

  ### 99-esxN (ESX N)

<details>
<summary>ESXi </summary>

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
</details>

## POD-199
<details>
<summary>POD-199 </summary>

 ### 199-lf01 (Leaf 01)

<details>
<summary>199-lf01 </summary>

```bash
configure terminal
hostname 199-lf01

feature udld
feature lacp
feature vpc
feature lldp
feature bfd
feature interface-vlan
feature vn-segment-vlan-based
feature ospf
feature bgp
feature nv overlay
nv overlay evpn
feature ngoam

clock timezone MSK 3 0

vlan 13
  name VLAN13
  vn-segment 19913
vlan 14
  name VLAN14
  vn-segment 19914
vlan 99
  name VLAN99
  vn-segment 19999

fabric forwarding anycast-gateway-mac 0000.0199.0001


vrf context VRF_13
  vni 19913
  rd auto
  address-family ipv4 unicast
    route-target import 65199:19913
    route-target import 65199:19913 evpn
    route-target export 65199:19913
    route-target export 65199:19913 evpn

vrf context VRF_14
  vni 19914
  rd auto
  address-family ipv4 unicast
    route-target import 65199:19914
    route-target import 65199:19914 evpn
    route-target export 65199:19914
    route-target export 65199:19914 evpn

interface mgmt0
  description OOB_Management
  vrf member management
  ip address 10.199.245.1/24

vpc domain 1
  role priority 200
  peer-keepalive destination 10.199.245.2 source 10.199.245.1 vrf management interval 1000 timeout 6 hold-timeout 3
  peer-gateway
  peer-switch
  auto-recovery reload-delay 600
  ip arp synchronize
  ipv6 nd synchronize

route-map PERMIT_VRF permit 10

udld aggressive

interface port-channel34
  description 199-lf02_Po34
  switchport
  switchport mode trunk
  switchport trunk allowed vlan all
  spanning-tree port type network
  vpc peer-link
  no shutdown


interface Ethernet1/3
  description to 199-lf02 Eth1/3 (Po34)
  switchport mode trunk
  switchport trunk allowed vlan all
  channel-group 34 mode active
  udld aggressive
  no shutdown

interface Ethernet1/4
  description to 199-lf02 Eth1/4 (Po34)
  switchport mode trunk
  switchport trunk allowed vlan all
  channel-group 34 mode active
  udld aggressive
  no shutdown

  interface Ethernet1/1
  description to 199-sp01 Eth1/1
  no switchport
  ip address 10.199.241.0/31
  no shutdown
  mtu 9216

interface Ethernet1/2
  description to 199-sp02 Eth1/1
  no switchport
  ip address 10.199.242.0/31
  no shutdown
  mtu 9216

interface port-channel5
  description to 199-esx1_Po1
  switchport
  switchport mode trunk
  switchport trunk allowed vlan 13,14,99
  spanning-tree port type edge trunk
  mtu 9216
  vpc 5
  no shutdown

interface Ethernet1/5
  description to 199-esx1_Eth1
  switchport mode trunk
  switchport trunk allowed vlan 13,14,99
  channel-group 5 mode active
  mtu 9216
  udld aggressive
  no shutdown

 interface port-channel6
  description to 199-esx2_Po1
  switchport
  switchport mode trunk
  switchport trunk allowed vlan 13,14,99
  spanning-tree port type edge trunk
  mtu 9216
  vpc 6
  no shutdown 

interface Ethernet1/6
  description to 199-esx2_Eth1
  switchport mode trunk
  switchport trunk allowed vlan 13,14,99
  mtu 9216
  channel-group 6 mode active
  no shutdown 

interface Vlan13
  description VLAN13
  mtu 9216
  no shutdown
  vrf member VRF_13
  no ip redirects
  ip address 192.168.13.254/24
  no ipv6 redirects
  fabric forwarding mode anycast-gateway

interface Vlan14
  description VLAN14
  mtu 9216
  no shutdown
  vrf member VRF_14
  no ip redirects
  ip address 192.168.14.254/24
  no ipv6 redirects
  fabric forwarding mode anycast-gateway

interface Vlan99
  description VLAN99
  mtu 9216
  no shutdown

interface loopback0
  description BGP_Router-ID
  ip address 10.199.243.1/32

interface loopback1
  description VTEP_Source
  ip address 10.199.244.1/32
  ip address 10.199.244.12/32 secondary

interface nve1
  no shutdown
  description VXLAN_Tunnel
  host-reachability protocol bgp
  advertise virtual-rmac
  source-interface loopback1
  member vni 19913 associate-vrf
  member vni 19914 associate-vrf
  member vni 19999
    ingress-replication protocol bgp


router bgp 65199
  router-id 10.199.243.1
  timers bgp 3 9
  bestpath as-path multipath-relax
  log-neighbor-changes
  address-family ipv4 unicast
    network 10.199.243.1/32
    network 10.199.244.1/32
    network 10.199.244.12/32
    maximum-paths 8
  address-family l2vpn evpn
    maximum-paths 8
    advertise-pip
  template peer SPINE-OVERLAY
    remote-as 65199
    update-source loopback0
    timers 6 18
    address-family l2vpn evpn
      send-community
      send-community extended
  template peer SPINE-UNDERLAY
    remote-as 65199
    timers 6 18
    address-family ipv4 unicast
      soft-reconfiguration inbound always
  neighbor 10.199.241.1
    inherit peer SPINE-UNDERLAY
  neighbor 10.199.242.1
    inherit peer SPINE-UNDERLAY
  neighbor 10.199.243.11
    inherit peer SPINE-OVERLAY
  neighbor 10.199.243.22
    inherit peer SPINE-OVERLAY
  vrf VRF_13
    log-neighbor-changes
    address-family ipv4 unicast
    redistribute direct route-map PERMIT_VRF
         maximum-paths 4
  vrf VRF_14
    log-neighbor-changes
    address-family ipv4 unicast
    redistribute direct route-map PERMIT_VRF
      maximum-paths 4
 ``` 
 
</details>

### 199-lf02 (Leaf 02)

<details>
<summary>199-lf02 </summary>

```bash
configure terminal
hostname 199-lf02

feature udld
feature lacp
feature vpc
feature lldp
feature bfd
feature interface-vlan
feature vn-segment-vlan-based
feature ospf
feature bgp
feature nv overlay
nv overlay evpn
feature ngoam

clock timezone MSK 3 0


vlan 13
  name VLAN13
  vn-segment 19913
vlan 14
  name VLAN14
  vn-segment 19914
vlan 99
  name VLAN99
  vn-segment 19999

fabric forwarding anycast-gateway-mac 0000.0199.0001


vrf context VRF_13
  vni 19913
  rd auto
  address-family ipv4 unicast
    route-target import 65199:19913
    route-target import 65199:19913 evpn
    route-target export 65199:19913
    route-target export 65199:19913 evpn

vrf context VRF_14
  vni 19914
  rd auto
  address-family ipv4 unicast
    route-target import 65199:19914
    route-target import 65199:19914 evpn
    route-target export 65199:19914
    route-target export 65199:19914 evpn

interface mgmt0
  description OOB_Management
  vrf member management
  ip address 10.199.245.2/24

vpc domain 1
  role priority 300
  peer-keepalive destination 10.199.245.1 source 10.199.245.2 vrf management interval 1000 timeout 6 hold-timeout 3 
  peer-gateway
  peer-switch
  auto-recovery reload-delay 600
  ip arp synchronize
  ipv6 nd synchronize

route-map PERMIT_VRF permit 10
udld aggressive

interface port-channel34
  description 199-lf01_Po34
  switchport
  switchport mode trunk
  switchport trunk allowed vlan all
  spanning-tree port type network
  vpc peer-link
  no shutdown

 interface Ethernet1/3
  description to 199-lf01 Eth1/3 (Po34)
  switchport mode trunk
  switchport trunk allowed vlan all
  channel-group 34 mode active
  udld aggressive
  no shutdown

interface Ethernet1/4
  description to 199-lf01 Eth1/4 (Po34)
  switchport mode trunk
  switchport trunk allowed vlan all
  channel-group 34 mode active
  udld aggressive
  no shutdown 

interface Ethernet1/1
  description to 199-sp01 Eth1/2
  no switchport
  ip address 10.199.241.2/31
  no shutdown

interface Ethernet1/2
  description to 199-sp02 Eth1/2
  no switchport
  ip address 10.199.242.2/31
  no shutdown

interface port-channel5
  description to 199-esx1_Po1
  switchport
  switchport mode trunk
  switchport trunk allowed vlan 13,14,99
  spanning-tree port type edge trunk
  mtu 9216
  vpc 5
  no shutdown

interface Ethernet1/5
  description to 199-esx1_Eth1
  switchport mode trunk
  switchport trunk allowed vlan 13,14,99
  channel-group 5 mode active
  mtu 9216
  udld aggressive
  no shutdown

 interface port-channel6
  description to 199-esx2_Po1
  switchport
  switchport mode trunk
  switchport trunk allowed vlan 13,14,99
  spanning-tree port type edge trunk
  mtu 9216
  vpc 6
  no shutdown 

 interface Ethernet1/6
  description to 199-esx2_Eth2
  switchport mode trunk
  switchport trunk allowed vlan 13,14,99
  mtu 9216
  channel-group 6 mode active
  no shutdown 

interface Vlan13
  description VLAN13
  mtu 9216
  no shutdown
  vrf member VRF_13
  no ip redirects
  ip address 192.168.13.254/24
  no ipv6 redirects
  fabric forwarding mode anycast-gateway

interface Vlan14
  description VLAN14
  mtu 9216
  no shutdown
  vrf member VRF_14
  no ip redirects
  ip address 192.168.14.254/24
  no ipv6 redirects
  fabric forwarding mode anycast-gateway

interface Vlan99
  description VLAN99
  mtu 9216
  no shutdown

interface loopback0
  description BGP_Router-ID
  ip address 10.199.243.2/32

interface loopback1
  description VTEP_Source
  ip address 10.199.244.2/32
  ip address 10.199.244.12/32 secondary


interface nve1
  no shutdown
  description VXLAN_Tunnel
  host-reachability protocol bgp
  advertise virtual-rmac
  source-interface loopback1
  member vni 19913 associate-vrf
  suppress-arp
  member vni 19914 associate-vrf
  suppress-arp
  member vni 19999
    ingress-replication protocol bgp

router bgp 65199
  router-id 10.199.243.2
  timers bgp 3 9
  bestpath as-path multipath-relax
  log-neighbor-changes
  address-family ipv4 unicast
    network 10.199.243.2/32
    network 10.199.244.2/32
    network 10.199.244.12/32
    maximum-paths 8
  address-family l2vpn evpn
    maximum-paths 8
    advertise-pip
  template peer SPINE-OVERLAY
    remote-as 65199
    update-source loopback0
    timers 6 18
    address-family l2vpn evpn
      send-community
      send-community extended
  template peer SPINE-UNDERLAY
    remote-as 65199
    timers 6 18
    address-family ipv4 unicast
      soft-reconfiguration inbound always
  neighbor 10.199.241.3
    inherit peer SPINE-UNDERLAY
  neighbor 10.199.242.3
    inherit peer SPINE-UNDERLAY
  neighbor 10.199.243.11
    inherit peer SPINE-OVERLAY
  neighbor 10.199.243.22
    inherit peer SPINE-OVERLAY
  vrf VRF_13
    log-neighbor-changes
    address-family ipv4 unicast
    redistribute direct route-map PERMIT_VRF
      maximum-paths 4
  vrf VRF_14
    log-neighbor-changes
    address-family ipv4 unicast
    redistribute direct route-map PERMIT_VRF
      maximum-paths 4

```
</details>

### 199-sp01 (Spine 01)

<details>
<summary>199-sp01 </summary>

```bash
configure terminal
hostname 199-sp01

feature udld
feature lldp
feature bfd
feature ospf
feature bgp
feature nv overlay
nv overlay evpn
feature ngoam

clock timezone MSK 3 0

route-map NEXTHOP permit 10
   set ip next-hop unchanged

interface mgmt0
  description OOB_Management
  vrf member management
  ip address 10.199.245.11/24

interface Ethernet1/1
  description to 199-lf01 Eth1/1
  no switchport
  ip address 10.199.241.1/31
  mtu 9216
  no shutdown

interface Ethernet1/2
  description to 199-lf02 Eth1/1
  no switchport
  ip address 10.199.241.3/31
  mtu 9216
  no shutdown

interface Ethernet1/3
  description to 199-bgw1 Eth1
  no switchport
  ip address 10.199.241.5/31
  mtu 9216
  no shutdown

interface Ethernet1/4
  description to 199-bgw2 Eth1
  no switchport
  ip address 10.199.241.7/31
  mtu 9216
  no shutdown

 interface loopback0
  description BGP_Router-ID
  ip address 10.199.243.11/32


 router bgp 65199
  router-id 10.199.243.11
  timers bgp 3 9
  bestpath as-path multipath-relax
  log-neighbor-changes

 address-family ipv4 unicast
    network 10.199.243.11/32
    maximum-paths 8

 address-family l2vpn evpn
    maximum-paths 8

  template peer UNDERLAY
    remote-as 65199
    timers 6 18
    address-family ipv4 unicast
      route-reflector-client
      next-hop-self
      soft-reconfiguration inbound always 

  template peer UNDERLAY-EBGP
    remote-as 65999
    timers 6 18
    address-family ipv4 unicast
    next-hop-self
      soft-reconfiguration inbound always 

 template peer OVERLAY
    remote-as 65199
    update-source loopback0
    timers 6 18
    address-family l2vpn evpn
    route-map NEXTHOP out
      route-reflector-client
      send-community
      send-community extended 

template peer OVERLAY-EBGP
    remote-as 65999
    update-source loopback0
    timers 6 18
    ebgp-multihop 10
    address-family l2vpn evpn
    route-map NEXTHOP out
      send-community
      send-community extended


 neighbor 10.199.241.0
    description 199-lf01-underlay
    inherit peer UNDERLAY
  
  neighbor 10.199.241.2
    description 199-lf02-underlay
    inherit peer UNDERLAY
  
  neighbor 10.199.241.4
    description 199-bgw1-underlay
    inherit peer UNDERLAY-EBGP
  
  neighbor 10.199.241.6
    description 199-bgw2-underlay
    inherit peer UNDERLAY-EBGP


   neighbor 10.199.243.1
    description 199-lf01-overlay
    inherit peer OVERLAY
  
   neighbor 10.199.243.2
    description 199-lf02-overlay
    inherit peer OVERLAY
  
   neighbor 10.199.243.33
    description 199-bgw1-overlay
    inherit peer OVERLAY-EBGP
  
  neighbor 10.199.243.44
    description 199-bgw2-overlay
    inherit peer OVERLAY-EBGP                  
```

</details>

### 199-sp02 (Spine 02)

<details>
<summary>199-sp02 </summary>

```bash 
configure terminal
hostname 199-sp02

feature udld
feature lldp
feature bfd
feature ospf
feature bgp
feature nv overlay
nv overlay evpn
feature ngoam

clock timezone MSK 3 0

route-map NEXTHOP permit 10
   set ip next-hop unchanged

interface mgmt0
  description OOB_Management
  vrf member management
  ip address 10.199.245.22/24

interface Ethernet1/1
  description to 199-lf01 Eth1/2
  no switchport
  ip address 10.199.242.1/31
  mtu 9216
  no shutdown

interface Ethernet1/2
  description to 199-lf02 Eth1/2
  no switchport
  ip address 10.199.242.3/31
  mtu 9216
  no shutdown

interface Ethernet1/3
  description to 199-bgw1 Eth2
  no switchport
  ip address 10.199.242.5/31
  mtu 9216
  no shutdown

interface Ethernet1/4
  description to 199-bgw2 Eth2
  no switchport
  ip address 10.199.242.7/31
  mtu 9216
  no shutdown

interface loopback0
  description BGP_Router-ID
  ip address 10.199.243.22/32


router bgp 65199
  router-id 10.199.243.22
  timers bgp 3 9
  bestpath as-path multipath-relax
  log-neighbor-changes

  address-family ipv4 unicast
    network 10.199.243.22/32
    maximum-paths 8
  address-family l2vpn evpn
      maximum-paths 8

 template peer UNDERLAY
    remote-as 65199
    timers 6 18
    address-family ipv4 unicast
      route-reflector-client
      next-hop-self
      soft-reconfiguration inbound always

    template peer UNDERLAY-EBGP
    remote-as 65999
    timers 6 18
    address-family ipv4 unicast
    next-hop-self
      soft-reconfiguration inbound always   

  template peer OVERLAY
    remote-as 65199
    update-source loopback0
    timers 6 18
    address-family l2vpn evpn
    route-map NEXTHOP out
      route-reflector-client
      send-community
      send-community extended
   
   template peer OVERLAY-EBGP
    remote-as 65999
    update-source loopback0
    timers 6 18
    address-family l2vpn evpn
    route-map NEXTHOP out
      send-community
      send-community extended

  neighbor 10.199.242.0
    description 199-lf01-underlay
    inherit peer UNDERLAY
  
  neighbor 10.199.242.2
    description 199-lf02-underlay
    inherit peer UNDERLAY
  
  neighbor 10.199.242.4
    description 199-bgw1-underlay
    inherit peer UNDERLAY-EBGP
  
  neighbor 10.199.242.6
    description 199-bgw2-underlay
    inherit peer UNDERLAY-EBGP
   
   neighbor 10.199.243.1
    description 199-lf01-overlay
    inherit peer OVERLAY

   neighbor 10.199.243.2
    description 199-lf02-overlay
    inherit peer OVERLAY

  neighbor 10.199.243.33
    description 199-bgw1-overlay
    inherit peer OVERLAY-EBGP
  
  neighbor 10.199.243.44
    description 199-bgw2-overlay
    inherit peer OVERLAY-EBGP            
```
</details>

### 199-bgw01 (BGW 01)

<details>
<summary>199-bgw01 </summary>

```bash
hostname 199-bgw1
!
spanning-tree mode mstp
!
vlan 99
   name VLAN99
!
vrf instance MGMT
!
vrf instance VRF_13
!
vrf instance VRF_14
!
interface Ethernet1
   description to-199-sp01-Eth1/3
   mtu 9214
   no switchport
   ip address 10.199.241.4/31
!
interface Ethernet2
   description to-199-sp02-Eth1/3
   mtu 9214
   no switchport
   ip address 10.199.242.4/31
!
interface Ethernet3
   description to-99-blf01-Eth6
   mtu 9214
   no switchport
   ip address 10.88.241.0/31
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Loopback0
   description BGP-Router-ID-POD199
   ip address 10.199.243.33/32
!
interface Loopback1
   description VTEP-Source-POD199
   ip address 10.199.244.33/32
!
interface Loopback10
   description BGP-Router-ID-DCI
   ip address 10.88.243.33/32
!
interface Loopback11
   description VTEP-Source-DCI
   ip address 10.88.244.33/32
!
interface Loopback13
   vrf VRF_13
   ip address 192.168.133.3/24
!
interface Loopback14
   vrf VRF_14
   ip address 192.168.144.3/24
!
interface Management1
   description OOB_Management
   vrf MGMT
   ip address 10.199.245.33/24
!
interface Vxlan1
   description VXLAN-Tunnel-Endpoint
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 99 vni 19999
   vxlan vrf VRF_13 vni 19913
   vxlan vrf VRF_14 vni 19914
   vxlan learn-restrict any
!
ip routing
ip routing vrf MGMT
ip routing vrf VRF_13
ip routing vrf VRF_14
!
ip prefix-list PL_BGW_TO_DCI seq 5 permit 10.88.243.33/32
ip prefix-list PL_BGW_TO_DCI seq 10 permit 10.88.244.33/32
ip prefix-list PL_BGW_TO_SPINE seq 5 permit 10.199.243.33/32
ip prefix-list PL_BGW_TO_SPINE seq 10 permit 10.199.244.33/32
!
route-map RM_BGW_TO_DCI permit 10
   match ip address prefix-list PL_BGW_TO_DCI
!
route-map RM_BGW_TO_SPINE permit 10
   match ip address prefix-list PL_BGW_TO_SPINE
!
route-map RM_EVPN_TO_DCI permit 10
   set ip next-hop 10.88.243.44
!
router bgp 65999
   router-id 10.199.243.33
   no bgp default ipv4-unicast
   timers bgp 3 9
   rd auto
   maximum-paths 10 ecmp 10
   neighbor DCI-EVPN peer group
   neighbor DCI-EVPN remote-as 65099
   neighbor DCI-EVPN update-source Loopback10
   neighbor DCI-EVPN ebgp-multihop 20
   neighbor DCI-EVPN send-community extended
   neighbor DCI-UNDERLAY peer group
   neighbor DCI-UNDERLAY remote-as 65099
   neighbor DCI-UNDERLAY ebgp-multihop 10
   neighbor DCI-UNDERLAY timers 6 18
   neighbor DCI-UNDERLAY send-community
   neighbor SPINE-EVPN peer group
   neighbor SPINE-EVPN remote-as 65199
   neighbor SPINE-EVPN update-source Loopback0
   neighbor SPINE-EVPN ebgp-multihop 10
   neighbor SPINE-EVPN send-community extended
   neighbor SPINE-EVPN maximum-routes 12000
   neighbor SPINE-UNDERLAY peer group
   neighbor SPINE-UNDERLAY remote-as 65199
   neighbor SPINE-UNDERLAY timers 3 9
   neighbor SPINE-UNDERLAY send-community
   neighbor 10.88.241.1 peer group DCI-UNDERLAY
   neighbor 10.88.243.1 peer group DCI-EVPN
   neighbor 10.199.241.5 peer group SPINE-UNDERLAY
   neighbor 10.199.242.5 peer group SPINE-UNDERLAY
   neighbor 10.199.243.11 peer group SPINE-EVPN
   neighbor 10.199.243.22 peer group SPINE-EVPN
   !
   vlan 99
      rd evpn domain all 10.88.243.33:99
      route-target both 65199:19999
      route-target import evpn domain remote 65999:19999
      route-target export evpn domain remote 65999:19999
      redistribute learned
   !
   address-family evpn
      neighbor DCI-EVPN activate
      neighbor DCI-EVPN domain remote
      neighbor SPINE-EVPN activate
      domain identifier 1:1
      neighbor default next-hop-self received-evpn-routes route-type ip-prefix inter-domain
   !
   address-family ipv4
      neighbor DCI-UNDERLAY activate
      neighbor SPINE-UNDERLAY activate
      neighbor SPINE-UNDERLAY route-map RM_BGW_TO_SPINE out
      network 10.88.243.33/32
      network 10.88.244.33/32
      network 10.199.243.33/32
      network 10.199.244.33/32
   !
   vrf VRF_13
      rd 65881:19913
      route-target import evpn 65199:19913
      route-target export evpn 65199:19913
      !
      address-family ipv4
         redistribute connected
   !
   vrf VRF_14
      rd 65881:19914
      route-target import evpn 65199:19914
      route-target export evpn 65199:19914
      !
      address-family ipv4
         redistribute connected
!
end

 ```

</details>

### 199-bgw02 (BGW 02)

<details>
<summary>199-bgw02 </summary>

```bash
hostname 199-bgw2
!
spanning-tree mode mstp
!
vlan 99
   name VLAN99
!
vrf instance MGMT
!
vrf instance VRF_13
!
vrf instance VRF_14
!
interface Ethernet1
   description to-199-sp01-Eth1/4
   mtu 9214
   no switchport
   ip address 10.199.241.6/31
!
interface Ethernet2
   description to-199-sp02-Eth1/4
   mtu 9214
   no switchport
   ip address 10.199.242.6/31
!
interface Ethernet3
   description to-99-blf02-Eth6
   mtu 9214
   no switchport
   ip address 10.88.242.0/31
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Ethernet9
!
interface Loopback0
   description BGP-Router-ID-POD199
   ip address 10.199.243.44/32
!
interface Loopback1
   description VTEP-Source-POD199
   ip address 10.199.244.44/32
!
interface Loopback10
   description BGP-Router-ID-DCI
   ip address 10.88.243.44/32
!
interface Loopback11
   description VTEP-Source-DCI
   ip address 10.88.244.44/32
!
interface Management1
   description OOB_Management
   vrf MGMT
   ip address 10.199.245.4/24
!
interface Vxlan1
   description VXLAN-Tunnel-Endpoint
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 99 vni 19999
   vxlan vrf VRF_13 vni 19913
   vxlan vrf VRF_14 vni 19914
   vxlan learn-restrict any
!
ip routing
ip routing vrf MGMT
ip routing vrf VRF_13
ip routing vrf VRF_14
!
ip prefix-list PL_BGW_TO_DCI seq 5 permit 10.88.243.44/32
ip prefix-list PL_BGW_TO_DCI seq 10 permit 10.88.244.44/32
ip prefix-list PL_BGW_TO_SPINE seq 5 permit 10.199.243.44/32
ip prefix-list PL_BGW_TO_SPINE seq 10 permit 10.199.244.44/32
!
route-map RM_BGW_TO_DCI permit 10
   match ip address prefix-list PL_BGW_TO_DCI
!
route-map RM_BGW_TO_SPINE permit 10
   match ip address prefix-list PL_BGW_TO_SPINE
!
router bgp 65999
   router-id 10.199.243.44
   no bgp default ipv4-unicast
   timers bgp 3 9
   rd auto
   maximum-paths 10 ecmp 10
   neighbor DCI-EVPN peer group
   neighbor DCI-EVPN remote-as 65099
   neighbor DCI-EVPN update-source Loopback10
   neighbor DCI-EVPN ebgp-multihop 2
   neighbor DCI-EVPN send-community extended
   neighbor DCI-UNDERLAY peer group
   neighbor DCI-UNDERLAY remote-as 65099
   neighbor DCI-UNDERLAY ebgp-multihop 10
   neighbor DCI-UNDERLAY timers 3 9
   neighbor DCI-UNDERLAY send-community
   neighbor SPINE-EVPN peer group
   neighbor SPINE-EVPN remote-as 65199
   neighbor SPINE-EVPN update-source Loopback0
   neighbor SPINE-EVPN ebgp-multihop 10
   neighbor SPINE-EVPN send-community extended
   neighbor SPINE-EVPN maximum-routes 12000
   neighbor SPINE-UNDERLAY peer group
   neighbor SPINE-UNDERLAY remote-as 65199
   neighbor SPINE-UNDERLAY timers 3 9
   neighbor SPINE-UNDERLAY send-community
   neighbor 10.88.242.1 peer group DCI-UNDERLAY
   neighbor 10.88.243.2 peer group DCI-EVPN
   neighbor 10.199.241.7 peer group SPINE-UNDERLAY
   neighbor 10.199.242.7 peer group SPINE-UNDERLAY
   neighbor 10.199.243.11 peer group SPINE-EVPN
   neighbor 10.199.243.22 peer group SPINE-EVPN
   !
   vlan 99
      rd evpn domain all 10.88.243.44:99
      route-target both 65199:19999
      route-target import evpn domain remote 65999:19999
      route-target export evpn domain remote 65999:19999
      redistribute learned
   !
   address-family evpn
      neighbor DCI-EVPN activate
      neighbor DCI-EVPN domain remote
      neighbor SPINE-EVPN activate
      domain identifier 1:1
      neighbor default next-hop-self received-evpn-routes route-type ip-prefix inter-domain
   !
   address-family ipv4
      neighbor DCI-UNDERLAY activate
      neighbor SPINE-UNDERLAY activate
      neighbor SPINE-UNDERLAY route-map RM_BGW_TO_SPINE out
      network 10.88.243.44/32
      network 10.88.244.44/32
      network 10.199.243.44/32
      network 10.199.244.44/32
   !
   vrf VRF_13
      rd 65882:19913
      route-target import evpn 65199:19913
      route-target export evpn 65199:19913
      !
      address-family ipv4
         redistribute connected
   !
   vrf VRF_14
      rd 65882:19914
      route-target import evpn 65199:19914
      route-target export evpn 65199:19914
      !
      address-family ipv4
         redistribute connected
!
end



```

</details>

</details>






---

## Проверка IP связности
 ## 1. Проверка маршрутной информации

 Проверим что на спайнах цода 199 и на блф все нужные сессии bgp подняты
 ```
 199-sp02# sh bgp l2vpn evpn summary
BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 10.199.243.22, local AS number 65199
BGP table version is 18890, L2VPN EVPN config peers 4, capable peers 4
22 network entries and 26 paths using 6056 bytes of memory
BGP attribute entries [18/3312], BGP AS path entries [3/30]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.199.243.1    4 65199   27677   27378    18890    0    0 01:05:17 5
10.199.243.2    4 65199   30537   30408    18890    0    0 00:36:40 3
10.199.243.33   4 65999   61549   50262    18890    0    0 00:00:11 10
10.199.243.44   4 65999   40012   33336    18890    0    0 00:00:18 8
199-sp02#
199-sp02# sh bgp ipv4 unicast summary
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 10.199.243.22, local AS number 65199
BGP table version is 6813, IPv4 Unicast config peers 4, capable peers 4
10 network entries and 11 paths using 2648 bytes of memory
BGP attribute entries [3/552], BGP AS path entries [1/6]
BGP community entries [0/0], BGP clusterlist entries [0/0]
10 received paths for inbound soft reconfiguration
10 identical, 0 modified, 0 filtered received paths using 0 bytes

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.199.242.0    4 65199   27424   27629     6813    0    0 00:36:50 3
10.199.242.2    4 65199   30442   30504     6813    0    0 00:36:50 3
10.199.242.4    4 65999   60544   52147     6813    0    0 00:00:18 2
10.199.242.6    4 65999   40759   34907     6813    0    0 00:00:27 2

99-blf1#sh bgp summary
BGP summary information for VRF default
Router identifier 10.99.243.1, local AS number 65099
Neighbor              AS Session State AFI/SAFI                AFI/SAFI State   NLRI Rcd   NLRI Acc
------------ ----------- ------------- ----------------------- -------------- ---------- ----------
10.88.241.0        65999 Established   IPv4 Unicast            Negotiated             11         11
10.88.243.33       65999 Established   L2VPN EVPN              Negotiated              5          5
10.99.241.1        65099 Established   IPv4 Unicast            Negotiated              9          9
10.99.242.1        65099 Established   IPv4 Unicast            Negotiated              9          9
10.99.243.11       65099 Established   L2VPN EVPN              Negotiated             49         49
10.99.243.22       65099 Established   L2VPN EVPN              Negotiated             49         49
10.99.246.2        65099 Established   IPv4 Unicast            Negotiated             16         16


```
Также глянем мак таблицу "растянутого" влана 99 

```
199-lf01# sh mac address-table vlan 99
Legend:
        * - primary entry, G - Gateway MAC, (R) - Routed MAC, O - Overlay MAC
        age - seconds since last seen,+ - primary entry using vPC Peer-Link,
        (T) - True, (F) - False, C - ControlPlane MAC, ~ - vsan
   VLAN     MAC Address      Type      age     Secure NTFY Ports
---------+-----------------+--------+---------+------+----+------------------
C   99     5000.00d5.5dc0   dynamic  0         F      F    nve1(10.199.244.33)
*   99     5000.00d8.ac19   dynamic  0         F      F    Po5
G   99     500d.0000.1b08   static   -         F      F    vPC Peer-Link(R)
G   99     5018.0000.1b08   static   -         F      F    sup-eth1(R)

```
И проверим маршруты в наших VRF 13 и 14 
```
199-lf01# sh ip route vrf VRF_13
IP Route Table for VRF "VRF_13"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

192.168.13.0/24, ubest/mbest: 1/0, attached
    *via 192.168.13.254, Vlan13, [0/0], 1d01h, direct
192.168.13.1/32, ubest/mbest: 1/0, attached
    *via 192.168.13.1, Vlan13, [190/0], 00:12:23, hmm
192.168.13.2/32, ubest/mbest: 1/0, attached
    *via 192.168.13.2, Vlan13, [190/0], 00:12:23, hmm
192.168.13.254/32, ubest/mbest: 1/0, attached
    *via 192.168.13.254, Vlan13, [0/0], 1d01h, local
192.168.133.3/32, ubest/mbest: 1/0
    *via 10.199.244.33%default, [200/0], 00:00:13, bgp-65199, internal, tag 6599
9, segid: 19913 tunnelid: 0xac7f421 encap: VXLAN

192.168.133.4/32, ubest/mbest: 1/0
    *via 10.199.244.44%default, [200/0], 00:00:13, bgp-65199, internal, tag 6599
9, segid: 19913 tunnelid: 0xac7f42c encap: VXLAN

192.168.134.3/32, ubest/mbest: 1/0
    *via 10.199.244.44%default, [200/0], 00:00:15, bgp-65199, internal, tag 6599
9, segid: 19913 tunnelid: 0xac7f42c encap: VXLAN

192.168.134.4/32, ubest/mbest: 1/0
    *via 10.199.244.33%default, [200/0], 00:00:13, bgp-65199, internal, tag 6599
9, segid: 19913 tunnelid: 0xac7f421 encap: VXLAN

192.168.233.0/24, ubest/mbest: 1/0
    *via 10.199.244.33%default, [200/0], 00:00:13, bgp-65199, internal, tag 6599
9, segid: 19913 tunnelid: 0xac7f421 encap: VXLAN


199-lf01# sh ip route vrf VRF_14
IP Route Table for VRF "VRF_14"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

0.0.0.0/0, ubest/mbest: 1/0
    *via 10.199.244.44%default, [200/0], 00:00:22, bgp-65199, internal, tag 6599
9, segid: 19914 tunnelid: 0xac7f42c encap: VXLAN

10.99.14.0/31, ubest/mbest: 1/0
    *via 10.199.244.44%default, [200/0], 00:00:31, bgp-65199, internal, tag 6599
9, segid: 19914 tunnelid: 0xac7f42c encap: VXLAN

10.99.14.2/31, ubest/mbest: 1/0
    *via 10.199.244.44%default, [200/0], 00:00:32, bgp-65199, internal, tag 6599
9, segid: 19914 tunnelid: 0xac7f42c encap: VXLAN

192.168.14.0/24, ubest/mbest: 1/0, attached
    *via 192.168.14.254, Vlan14, [0/0], 1d01h, direct
192.168.14.1/32, ubest/mbest: 1/0, attached
    *via 192.168.14.1, Vlan14, [190/0], 00:12:39, hmm
192.168.14.2/32, ubest/mbest: 1/0, attached
    *via 192.168.14.2, Vlan14, [190/0], 00:12:39, hmm
192.168.14.254/32, ubest/mbest: 1/0, attached
    *via 192.168.14.254, Vlan14, [0/0], 1d01h, local
192.168.144.3/32, ubest/mbest: 1/0
    *via 10.199.244.33%default, [200/0], 00:00:29, bgp-65199, internal, tag 6599
9, segid: 19914 tunnelid: 0xac7f421 encap: VXLAN

192.168.144.4/32, ubest/mbest: 1/0
    *via 10.199.244.44%default, [200/0], 00:00:29, bgp-65199, internal, tag 6599
9, segid: 19914 tunnelid: 0xac7f42c encap: VXLAN

192.168.244.0/24, ubest/mbest: 1/0
    *via 10.199.244.44%default, [200/0], 00:00:31, bgp-65199, internal, tag 6599
9, segid: 19914 tunnelid: 0xac7f42c encap: VXLAN

```
Видим что во влане 99 связность есть, в VRF 13 видим маршруты лупбеков из другого цода 192.168.133.3/32 192.168.134.4/32 через разные БГВ
В VRF 14 видим прилетающий с фаерволлов дефолт и маршруты до p2p интерфейсов с фаерволлами и лупбеков с хостами из обоих цодов


## 2. Проверка межсерверной связности между VM 


### 2.1. Проверка связности в Vlan 99
| Device Name | Network Address |
|-------------|-------------------|
| 199-esx1    | 192.168.99.1/24   |
| 99-esx1     | 192.168.99.11/24  |
| 99-esx4     | 192.168.99.44/24  |

```
199-esx1#ping vrf VRF_99 192.168.99.11
PING 192.168.99.11 (192.168.99.11) 72(100) bytes of data.
80 bytes from 192.168.99.11: icmp_seq=1 ttl=64 time=25.1 ms
80 bytes from 192.168.99.11: icmp_seq=2 ttl=64 time=23.5 ms
80 bytes from 192.168.99.11: icmp_seq=3 ttl=64 time=18.5 ms
80 bytes from 192.168.99.11: icmp_seq=4 ttl=64 time=36.9 ms
80 bytes from 192.168.99.11: icmp_seq=5 ttl=64 time=28.6 ms

--- 192.168.99.11 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 85ms
rtt min/avg/max/mdev = 18.581/26.560/36.913/6.102 ms, pipe 2, ipg/ewma 21.286/26.104 ms
199-esx1#
199-esx1#ping vrf VRF_99 192.168.99.44
PING 192.168.99.44 (192.168.99.44) 72(100) bytes of data.
80 bytes from 192.168.99.44: icmp_seq=1 ttl=64 time=49.4 ms
80 bytes from 192.168.99.44: icmp_seq=2 ttl=64 time=50.2 ms
80 bytes from 192.168.99.44: icmp_seq=3 ttl=64 time=40.9 ms
80 bytes from 192.168.99.44: icmp_seq=4 ttl=64 time=37.6 ms
80 bytes from 192.168.99.44: icmp_seq=5 ttl=64 time=30.8 ms

--- 192.168.99.44 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 43ms
rtt min/avg/max/mdev = 30.853/41.837/50.245/7.316 ms, pipe 5, ipg/ewma 10.964/45.102 ms
```

### 2.2. Проверка связности в VRF_13

| Device Name | Network Address |
|-------------|-------------------|
| 199-esx1    | 192.168.13.1/24   |
| 199-lf01/02 | 192.168.13.254/24 |
| 99-lf04 lo13| 192.168.134.4/32  |
| 199-bgw1    | 192.168.133.3/32  |

```

199-esx1#
199-esx1#ping vrf VRF_13 192.168.13.254
PING 192.168.13.254 (192.168.13.254) 72(100) bytes of data.
80 bytes from 192.168.13.254: icmp_seq=1 ttl=255 time=17.6 ms
80 bytes from 192.168.13.254: icmp_seq=2 ttl=255 time=4.53 ms
80 bytes from 192.168.13.254: icmp_seq=3 ttl=255 time=5.40 ms
80 bytes from 192.168.13.254: icmp_seq=4 ttl=255 time=40.1 ms

--- 192.168.13.254 ping statistics ---
5 packets transmitted, 4 received, 20% packet loss, time 67ms
rtt min/avg/max/mdev = 4.530/16.924/40.128/14.364 ms, pipe 2, ipg/ewma 16.850/17.855 ms
199-esx1#
199-esx1#ping vrf VRF_13 192.168.134.4
PING 192.168.134.4 (192.168.134.4) 72(100) bytes of data.
80 bytes from 192.168.134.4: icmp_seq=1 ttl=61 time=71.8 ms
80 bytes from 192.168.134.4: icmp_seq=2 ttl=61 time=61.0 ms
80 bytes from 192.168.134.4: icmp_seq=3 ttl=61 time=53.1 ms
80 bytes from 192.168.134.4: icmp_seq=4 ttl=61 time=45.1 ms
80 bytes from 192.168.134.4: icmp_seq=5 ttl=61 time=37.6 ms

--- 192.168.134.4 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 44ms
rtt min/avg/max/mdev = 37.603/53.756/71.859/11.963 ms, pipe 5, ipg/ewma 11.093/61.958 ms
199-esx1#
199-esx1#ping vrf VRF_13 192.168.133.3
PING 192.168.133.3 (192.168.133.3) 72(100) bytes of data.
80 bytes from 192.168.133.3: icmp_seq=1 ttl=63 time=17.4 ms
80 bytes from 192.168.133.3: icmp_seq=2 ttl=63 time=11.4 ms
80 bytes from 192.168.133.3: icmp_seq=3 ttl=63 time=10.6 ms
80 bytes from 192.168.133.3: icmp_seq=4 ttl=63 time=39.3 ms
80 bytes from 192.168.133.3: icmp_seq=5 ttl=63 time=31.0 ms

--- 192.168.133.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 62ms
rtt min/avg/max/mdev = 10.607/21.976/39.333/11.352 ms, pipe 2, ipg/ewma 15.738/20.390 ms

199-esx1#tracer vrf VRF_13 192.168.134.4
traceroute to 192.168.134.4 (192.168.134.4), 30 hops max, 60 byte packets
 1  _gateway (192.168.13.254)  6.178 ms  5.016 ms  31.834 ms
 2  192.168.133.3 (192.168.133.3)  33.280 ms  37.753 ms  39.276 ms
 3  192.168.233.11 (192.168.233.11)  57.508 ms  60.184 ms  64.006 ms
 4  192.168.134.4 (192.168.134.4)  85.399 ms  88.140 ms  93.686 ms

```
------------------------------------------------------------------------------------------------------

### 2.3. Проверка связности в VRF_14

| Device Name | Network Address   | VRF Name   |
|-------------|-------------------|------------|
| 199-esx1    | 192.168.14.1/24   | VRF_14     |
| 199-lf01/02 | 192.168.14.254/24 | VRF_14     |
| 99-esx4     | 192.168.4.1/24    | VRF_CORE_4 |
| 99-esx1     | 192.168.1.1/24    | VRF_CORE_1 |
| 199-bgw2 LO14| 192.168.144.4/32 | VRF_14     |
| 99-lf01 p2p fw01| 10.99.14.1/31  | VRF_14    |

```
199-esx1#ping vrf VRF_14 192.168.14.254
PING 192.168.14.254 (192.168.14.254) 72(100) bytes of data.
80 bytes from 192.168.14.254: icmp_seq=1 ttl=255 time=38.1 ms
80 bytes from 192.168.14.254: icmp_seq=2 ttl=255 time=30.5 ms
80 bytes from 192.168.14.254: icmp_seq=3 ttl=255 time=24.0 ms
80 bytes from 192.168.14.254: icmp_seq=4 ttl=255 time=31.7 ms
80 bytes from 192.168.14.254: icmp_seq=5 ttl=255 time=5.63 ms

--- 192.168.14.254 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 72ms
rtt min/avg/max/mdev = 5.631/26.029/38.188/11.138 ms, pipe 4, ipg/ewma 18.149/31.419 ms
199-esx1#
199-esx1#ping vrf VRF_14 192.168.4.1
PING 192.168.4.1 (192.168.4.1) 72(100) bytes of data.
80 bytes from 192.168.4.1: icmp_seq=1 ttl=57 time=167 ms
80 bytes from 192.168.4.1: icmp_seq=2 ttl=57 time=156 ms
80 bytes from 192.168.4.1: icmp_seq=3 ttl=57 time=148 ms
80 bytes from 192.168.4.1: icmp_seq=4 ttl=57 time=153 ms
80 bytes from 192.168.4.1: icmp_seq=5 ttl=57 time=146 ms

--- 192.168.4.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 43ms
rtt min/avg/max/mdev = 146.055/154.562/167.779/7.599 ms, pipe 5, ipg/ewma 10.989/160.751 ms
199-esx1#
199-esx1#ping vrf VRF_14 192.168.1.1
PING 192.168.1.1 (192.168.1.1) 72(100) bytes of data.
80 bytes from 192.168.1.1: icmp_seq=1 ttl=59 time=35.8 ms
80 bytes from 192.168.1.1: icmp_seq=2 ttl=59 time=32.4 ms
80 bytes from 192.168.1.1: icmp_seq=3 ttl=59 time=28.0 ms
80 bytes from 192.168.1.1: icmp_seq=4 ttl=59 time=32.7 ms
80 bytes from 192.168.1.1: icmp_seq=5 ttl=59 time=29.1 ms

--- 192.168.1.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 92ms
rtt min/avg/max/mdev = 28.082/31.657/35.801/2.765 ms, pipe 3, ipg/ewma 23.226/33.623 ms

199-esx1#
199-esx1#ping vrf VRF_14 192.168.144.4
PING 192.168.144.4 (192.168.144.4) 72(100) bytes of data.
80 bytes from 192.168.144.4: icmp_seq=1 ttl=63 time=15.5 ms
80 bytes from 192.168.144.4: icmp_seq=2 ttl=63 time=11.0 ms
80 bytes from 192.168.144.4: icmp_seq=3 ttl=63 time=9.46 ms
80 bytes from 192.168.144.4: icmp_seq=4 ttl=63 time=11.5 ms
80 bytes from 192.168.144.4: icmp_seq=5 ttl=63 time=11.6 ms

--- 192.168.144.4 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 55ms
rtt min/avg/max/mdev = 9.463/11.853/15.532/2.000 ms, pipe 2, ipg/ewma 13.769/13.656 ms
199-esx1#
199-esx1#ping vrf VRF_14 10.99.14.1
PING 10.99.14.1 (10.99.14.1) 72(100) bytes of data.
80 bytes from 10.99.14.1: icmp_seq=1 ttl=62 time=28.4 ms
80 bytes from 10.99.14.1: icmp_seq=2 ttl=62 time=19.9 ms
80 bytes from 10.99.14.1: icmp_seq=3 ttl=62 time=19.1 ms
80 bytes from 10.99.14.1: icmp_seq=4 ttl=62 time=20.4 ms
80 bytes from 10.99.14.1: icmp_seq=5 ttl=62 time=23.4 ms

--- 10.99.14.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 76ms
rtt min/avg/max/mdev = 19.147/22.261/28.401/3.402 ms, pipe 3, ipg/ewma 19.015/25.310 ms


199-esx1#tracer vrf VRF_14 192.168.4.1
traceroute to 192.168.4.1 (192.168.4.1), 30 hops max, 60 byte packets
 1  _gateway (192.168.14.254)  26.303 ms  35.961 ms  36.441 ms         AG 199-lf01/02
 2  192.168.144.4 (192.168.144.4)  37.824 ms  46.714 ms  48.555 ms     199-bgw02
 3  10.99.14.2 (10.99.14.2)  54.965 ms  58.040 ms  65.082 ms           99-blf02
 4  10.99.14.1 (10.99.14.1)  67.344 ms  73.256 ms  75.254 ms           99-blf01
 5  * * *                                                              99-fw01
 6  10.99.4.1 (10.99.4.1)  99.384 ms  96.924 ms  98.445 ms             99-blf01
 7  192.168.4.254 (192.168.4.254)  150.413 ms  148.469 ms  144.229 ms  AG 99-lf03/04
 8  192.168.4.1 (192.168.4.1)  167.248 ms  160.558 ms  186.858 ms      99-esx4


```

## Итог
1. В ходе Проектной работы было настроено два полноценных POD с использованием оборудования разных вендоров Huawei, Arista, Nexus.
2. Были протестированы разные методики резервирования и отказоустойчивости каналов VPC MLAG Multihoming. 
3. Настроены разные типы связности через vxlan - L2 mac-vrf растянутый на обе площадки. L3 - разные типы VRF  c терминацией на FW и без.
4. Собран отказоустойчивый кластер Huawei USG 6000 Active/Standby.
