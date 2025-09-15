# Построение Underlay сети(BGP)

### Цели:
- ##### Настроить Overlay на основе VxLAN EVPN для L2 связанности между клиентами.

### Описание/Пошаговая инструкция выполнения домашнего задания:
- ##### Настроить BGP peering между Leaf и Spine в AF l2vpn evpn
- ##### Настроить связанность между клиентами в первой зоне и убедитесь в её наличии
- ##### Зафиксировать в документации - план работы, адресное пространство, схему сети, конфигурацию устройств

![Схема сети с адресацией](images/image1.png)

### Для IPv4
#### Адреса loopback-интерфейсов 
| Узел | Loopback-адрес |
|---------|--------|
| Spine-1 | 10.0.1.1 |
| Spine-2 | 10.0.2.2 |
| Leaf-1 | 10.0.0.1 |
| Leaf-2 | 10.0.0.2 |
| Leaf-3 | 10.0.0.3 |

##### Общая таблица связей:
| Связь | Подсеть | Адрес на Spine           | Адрес на Leaf           |
|---------------|---------------|---------------|---------------|
| Spine-1 <-> Leaf-1 | 172.16.1.0/30 | 172.16.1.1 | 172.16.1.2 |
| Spine-1 <-> Leaf-2 | 172.16.2.0/30 | 172.16.2.1 | 172.16.2.2 |
| Spine-1 <-> Leaf-3 | 172.16.3.0/30 | 172.16.3.1 | 172.16.3.2 |
| Spine-2 <-> Leaf-1 | 172.16.1.4/30 | 172.16.1.5 | 172.16.1.6 |
| Spine-2 <-> Leaf-2 | 172.16.2.4/30 | 172.16.2.5 | 172.16.2.6 |
| Spine-2 <-> Leaf-3 | 172.16.3.4/30 | 172.16.3.5 | 172.16.3.6 |

### Для IPv6
##### Адреса loopback-интерфейсов 
| Узел | Loopback-адрес |
|---------|--------|
| Spine-1	| 2001:db8:fab:ffff::1/128 |
| Spine-2       | 2001:db8:fab:ffff::2/128 |
| Leaf-1	| 2001:db8:fab:ffff::101/128 |
| Leaf-2	| 2001:db8:fab:ffff::102/128 |
| Leaf-3	| 2001:db8:fab:ffff::103/128 |

##### Общая таблица связей:
| Связь         | Подсеть                     | Адрес на Spine           | Адрес на Leaf           |
|---------------|-----------------------------|--------------------------|-------------------------|
| Spine1 <-> Leaf1  | 2001:db8:fab:1001::/64    | 2001:db8:fab:1001::1   | 2001:db8:fab:1001::2  |
| Spine1 <-> Leaf2  | 2001:db8:fab:1002::/64    | 2001:db8:fab:1002::1   | 2001:db8:fab:1002::2  |
| Spine1 <-> Leaf3  | 2001:db8:fab:1003::/64    | 2001:db8:fab:1003::1   | 2001:db8:fab:1003::2  |
| Spine2 <-> Leaf1  | 2001:db8:fab:2001::/64    | 2001:db8:fab:2001::1   | 2001:db8:fab:2001::2  |
| Spine2 <-> Leaf2  | 2001:db8:fab:2002::/64    | 2001:db8:fab:2002::1   | 2001:db8:fab:2002::2  |
| Spine2 <-> Leaf3  | 2001:db8:fab:2003::/64    | 2001:db8:fab:2003::1   | 2001:db8:fab:2003::2  |

##### Пример настроек:

Пример настроек со Spine-1:
```cfg
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname Spine-1
!
spanning-tree mode mstp
!
interface Ethernet1
   description "to-L1"
   no switchport
   ip address 172.16.1.1/30
   ipv6 address 2001:db8:fab:1001::1/64
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf area 0.0.0.1
   ipv6 ospf bfd
   ipv6 ospf network point-to-point
   ipv6 ospf 1 area 0.0.0.1
   no isis bfd
!
interface Ethernet2
   description "to-L2"
   no switchport
   ip address 172.16.2.1/30
   ipv6 address 2001:db8:fab:1002::1/64
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf area 0.0.0.1
   ipv6 ospf bfd
   ipv6 ospf network point-to-point
   ipv6 ospf 1 area 0.0.0.1
   no isis bfd
!
interface Ethernet3
   description "to-L3"
   no switchport
   ip address 172.16.3.1/30
   ipv6 address 2001:db8:fab:1003::1/64
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf area 0.0.0.1
   ipv6 ospf bfd
   ipv6 ospf network point-to-point
   ipv6 ospf 1 area 0.0.0.1
   no isis bfd
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
   ip address 10.0.1.1/32
   ipv6 address 2001:db8:fab:ffff::1/128
   ip ospf area 0.0.0.1
   ipv6 ospf 1 area 0.0.0.1
!
interface Management1
!
ip routing
!
ipv6 unicast-routing
!
router bgp 65100
   router-id 10.0.1.1
   timers bgp 3 9
   neighbor EVPN peer group
   neighbor EVPN remote-as 65100
   neighbor EVPN out-delay 0
   neighbor EVPN update-source Loopback0
   neighbor EVPN bfd
   neighbor EVPN route-reflector-client
   neighbor EVPN send-community extended
   neighbor 10.0.0.1 peer group EVPN
   neighbor 10.0.0.2 peer group EVPN
   neighbor 10.0.0.3 peer group EVPN
   !
   address-family evpn
      neighbor 10.0.0.1 activate
      neighbor 10.0.0.2 activate
      neighbor 10.0.0.3 activate
!
router ospf 1
   router-id 10.0.1.1
   area 0.0.0.1 stub no-summary
   max-lsa 12000
!
ipv6 router ospf 1
   router-id 10.0.1.1
   area 0.0.0.1 stub no-summary
!
end
```

Пример настроек с Leaf-1:
```cfg
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname Leaf-1
!
spanning-tree mode mstp
!
vlan 10
   name vl10
!
interface Ethernet1
   description "to-S1"
   no switchport
   ip address 172.16.1.2/30
   ipv6 address 2001:db8:fab:1001::2/64
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf area 0.0.0.1
   ipv6 ospf bfd
   ipv6 ospf network point-to-point
   ipv6 ospf 1 area 0.0.0.1
   no isis bfd
!
interface Ethernet2
   description "to-S2"
   no switchport
   ip address 172.16.1.6/30
   ipv6 address 2001:db8:fab:2001::2/64
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf area 0.0.0.1
   ipv6 ospf bfd
   ipv6 ospf network point-to-point
   ipv6 ospf 1 area 0.0.0.1
   no isis bfd
!
interface Ethernet3
   no ip ospf neighbor bfd
   no ipv6 ospf bfd
   no isis bfd
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
   switchport access vlan 10
!
interface Loopback0
   ip address 10.0.0.1/32
   ipv6 address 2001:db8:fab:ffff::101/128
   ip ospf area 0.0.0.1
   ipv6 ospf 1 area 0.0.0.1
!
interface Management1
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
!
ip routing
!
ipv6 unicast-routing
!
router bgp 65100
   router-id 10.0.0.1
   timers bgp 3 9
   maximum-paths 2 ecmp 2
   neighbor EVPN peer group
   neighbor EVPN remote-as 65100
   neighbor EVPN out-delay 0
   neighbor EVPN update-source Loopback0
   neighbor EVPN bfd
   neighbor EVPN send-community extended
   neighbor 10.0.1.1 peer group EVPN
   neighbor 10.0.2.2 peer group EVPN
   !
   vlan 10
      rd auto
      route-target both 65100:10010
      redistribute learned
   !
   address-family evpn
      neighbor 10.0.1.1 activate
      neighbor 10.0.2.2 activate
!
router ospf 1
   router-id 10.0.0.1
   area 0.0.0.1 stub
   max-lsa 12000
!
ipv6 router ospf 1
   router-id 10.0.0.1
   area 0.0.0.1 stub
!
end
```

#### Проверка

```cfg
Leaf-1#sh vxlan vtep
Remote VTEPS for Vxlan1:

VTEP           Tunnel Type(s)
-------------- --------------
10.0.0.2       unicast, flood

Total number of remote VTEPS:  1
Leaf-1#show bgp evpn summary 
BGP summary information for VRF default
Router identifier 10.0.0.1, local AS number 65100
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.0.1.1 4 65100            739       767    0    0 00:15:18 Estab   2      2
  10.0.2.2 4 65100            738       779    0    0 00:06:37 Estab   2      2
Leaf-1#show bgp evpn
BGP routing table information for VRF default
Router identifier 10.0.0.1, local AS number 65100
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.0.0.1:10 mac-ip 0050.7966.6806
                                 -                     -       -       0       i
 * >Ec    RD: 10.0.0.2:10 mac-ip 0050.7966.6807
                                 10.0.0.2              -       100     0       i Or-ID: 10.0.0.2 C-LST: 10.0.1.1 
 *  ec    RD: 10.0.0.2:10 mac-ip 0050.7966.6807
                                 10.0.0.2              -       100     0       i Or-ID: 10.0.0.2 C-LST: 10.0.2.2 
 * >      RD: 10.0.0.1:10 imet 10.0.0.1
                                 -                     -       -       0       i
 * >Ec    RD: 10.0.0.2:10 imet 10.0.0.2
                                 10.0.0.2              -       100     0       i Or-ID: 10.0.0.2 C-LST: 10.0.1.1 
 *  ec    RD: 10.0.0.2:10 imet 10.0.0.2
                                 10.0.0.2              -       100     0       i Or-ID: 10.0.0.2 C-LST: 10.0.2.2 

```

VPCs PING:
```
VPC1> sh 

NAME   IP/MASK              GATEWAY                             GATEWAY
VPC1   192.168.10.1/24      192.168.10.254
       fe80::250:79ff:fe66:6806/64

VPC1> ping 192.168.10.2

84 bytes from 192.168.10.2 icmp_seq=1 ttl=64 time=59.280 ms
84 bytes from 192.168.10.2 icmp_seq=2 ttl=64 time=57.593 ms
84 bytes from 192.168.10.2 icmp_seq=3 ttl=64 time=44.621 ms
84 bytes from 192.168.10.2 icmp_seq=4 ttl=64 time=44.735 ms
84 bytes from 192.168.10.2 icmp_seq=5 ttl=64 time=73.135 ms

VPC2> sh

NAME   IP/MASK              GATEWAY                             GATEWAY
VPC2   192.168.10.2/24      192.168.10.254
       fe80::250:79ff:fe66:6807/64

VPC2> ping 192.168.10.1

84 bytes from 192.168.10.1 icmp_seq=1 ttl=64 time=46.444 ms
84 bytes from 192.168.10.1 icmp_seq=2 ttl=64 time=50.814 ms
84 bytes from 192.168.10.1 icmp_seq=3 ttl=64 time=52.966 ms
84 bytes from 192.168.10.1 icmp_seq=4 ttl=64 time=39.294 ms
84 bytes from 192.168.10.1 icmp_seq=5 ttl=64 time=35.699 ms
```

### Конфиги устройств:
- [Spine-1](configs/S1.txt)
- [Spine-2](configs/S2.txt)
- [Leaf-1](configs/L1.txt)
- [Leaf-2](configs/L2.txt)
- [Leaf-3](configs/L3.txt)
