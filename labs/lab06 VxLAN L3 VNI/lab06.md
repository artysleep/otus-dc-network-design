# # VxLAN. EVPN L3

### Цели:
- ##### Настроить маршрутизацию в рамках Overlay между клиентами.




### Описание/Пошаговая инструкция выполнения домашнего задания:
- ##### Настроить каждого клиента в своем VNI
- ##### Настроить маршрутизацию между клиентами.
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

#### Адреса хостов
| Хост | Адрес | Шлюз | Vlan | VNI | 
|---------------|---------------|---------------|---------------|---------------|
| VPC1 | 192.168.10.1/24 | 192.168.10.254 | 10 | 10010 |
| VPC1 | 192.168.20.1/24 | 192.168.20.254 | 20 | 10020 |
| VPC1 | 192.168.30.1/24 | 192.168.30.254 | 30 | 10030 |
| VPC1 | 192.168.40.1/24 | 192.168.40.254 | 40 | 10040 |

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

Пример настроек со Leaf-1:
```cfg
vlan 10
   name vl10

vrf instance VRF1

interface Ethernet8
   switchport access vlan 10

interface Vlan10
   vrf VRF1
   ip address virtual 192.168.10.254/24

interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vrf VRF1 vni 100999

ip routing
ip routing vrf VRF1


router bgp 65100
   router-id 10.0.0.1
   no bgp default ipv4-unicast
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
      neighbor EVPN activate
   !
   vrf VRF1
      rd 1:100999
      route-target import evpn 65100:100999
      route-target export evpn 65100:100999

```

Пример настроек с Leaf-2:
```cfg
vrf instance VRF1

ip routing vrf VRF1

interface Vlan20
   vrf VRF1
   ip address virtual 192.168.20.254/24

interface Vxlan1
vxlan vrf VRF1 vni 100999

router bgp 65100
vrf VRF1
      rd 2:100999
      route-target import evpn 65100:100999
      route-target export evpn 65100:100999
```

Пример настроек с Leaf-3:
```cfg
vrf instance VRF1

ip routing vrf VRF1
	  
interface Vlan30
   vrf VRF1
   ip address virtual 192.168.30.254/24
   
interface Vlan40
   vrf VRF1
   ip address virtual 192.168.40.254/24

interface Vxlan1
vxlan vrf VRF1 vni 100999

router bgp 65100
vrf VRF1
      rd 3:100999
      route-target import evpn 65100:100999
      route-target export evpn 65100:100999
```

#### Проверка

```cfg
Leaf-1(config-if-Vl10)#show vxlan vtep
Remote VTEPS for Vxlan1:

VTEP           Tunnel Type(s)
-------------- --------------
10.0.0.2       unicast
10.0.0.3       unicast

Leaf-1(config-if-Vl10)#sh bgp evpn summary
BGP summary information for VRF default
Router identifier 10.0.0.1, local AS number 65100
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.0.1.1 4 65100          41789     41797    0    0 00:00:06 Estab   0      0
  10.0.2.2 4 65100          41740     41833    0   38 00:00:00 Estab   0      0

Leaf-1(config)#show bgp evpn route-type imet
BGP routing table information for VRF default
Router identifier 10.0.0.1, local AS number 65100
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.0.0.1:10 imet 10.0.0.1
                                 -                     -       -       0       i
 * >Ec    RD: 10.0.0.2:20 imet 10.0.0.2
                                 10.0.0.2              -       100     0       i Or-ID: 10.0.0.2 C-LST: 10.0.1.1
 *  ec    RD: 10.0.0.2:20 imet 10.0.0.2
                                 10.0.0.2              -       100     0       i Or-ID: 10.0.0.2 C-LST: 10.0.2.2
 * >Ec    RD: 10.0.0.3:30 imet 10.0.0.3
                                 10.0.0.3              -       100     0       i Or-ID: 10.0.0.3 C-LST: 10.0.1.1
 *  ec    RD: 10.0.0.3:30 imet 10.0.0.3
                                 10.0.0.3              -       100     0       i Or-ID: 10.0.0.3 C-LST: 10.0.2.2
 * >Ec    RD: 10.0.0.3:40 imet 10.0.0.3
                                 10.0.0.3              -       100     0       i Or-ID: 10.0.0.3 C-LST: 10.0.1.1
 *  ec    RD: 10.0.0.3:40 imet 10.0.0.3
                                 10.0.0.3              -       100     0       i Or-ID: 10.0.0.3 C-LST: 10.0.2.2
Leaf-1(config)#show bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 10.0.0.1, local AS number 65100
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.0.0.1:10 mac-ip 0050.7966.6806
                                 -                     -       -       0       i
 * >      RD: 10.0.0.1:10 mac-ip 0050.7966.6806 192.168.10.1
                                 -                     -       -       0       i
 * >Ec    RD: 10.0.0.2:20 mac-ip 0050.7966.6807
                                 10.0.0.2              -       100     0       i Or-ID: 10.0.0.2 C-LST: 10.0.2.2
 *  ec    RD: 10.0.0.2:20 mac-ip 0050.7966.6807
                                 10.0.0.2              -       100     0       i Or-ID: 10.0.0.2 C-LST: 10.0.1.1
 * >Ec    RD: 10.0.0.2:20 mac-ip 0050.7966.6807 192.168.20.1
                                 10.0.0.2              -       100     0       i Or-ID: 10.0.0.2 C-LST: 10.0.2.2
 *  ec    RD: 10.0.0.2:20 mac-ip 0050.7966.6807 192.168.20.1
                                 10.0.0.2              -       100     0       i Or-ID: 10.0.0.2 C-LST: 10.0.1.1
 * >Ec    RD: 10.0.0.3:30 mac-ip 0050.7966.6808
                                 10.0.0.3              -       100     0       i Or-ID: 10.0.0.3 C-LST: 10.0.2.2
 *  ec    RD: 10.0.0.3:30 mac-ip 0050.7966.6808
                                 10.0.0.3              -       100     0       i Or-ID: 10.0.0.3 C-LST: 10.0.1.1
 * >Ec    RD: 10.0.0.3:30 mac-ip 0050.7966.6808 192.168.30.1
                                 10.0.0.3              -       100     0       i Or-ID: 10.0.0.3 C-LST: 10.0.2.2
 *  ec    RD: 10.0.0.3:30 mac-ip 0050.7966.6808 192.168.30.1
                                 10.0.0.3              -       100     0       i Or-ID: 10.0.0.3 C-LST: 10.0.1.1
 * >Ec    RD: 10.0.0.3:40 mac-ip 0050.7966.6809
                                 10.0.0.3              -       100     0       i Or-ID: 10.0.0.3 C-LST: 10.0.1.1
 *  ec    RD: 10.0.0.3:40 mac-ip 0050.7966.6809
                                 10.0.0.3              -       100     0       i Or-ID: 10.0.0.3 C-LST: 10.0.2.2
 * >Ec    RD: 10.0.0.3:40 mac-ip 0050.7966.6809 192.168.40.1
                                 10.0.0.3              -       100     0       i Or-ID: 10.0.0.3 C-LST: 10.0.1.1
 *  ec    RD: 10.0.0.3:40 mac-ip 0050.7966.6809 192.168.40.1
                                 10.0.0.3              -       100     0       i Or-ID: 10.0.0.3 C-LST: 10.0.2.2

Leaf-1(config)#show ip route vrf VRF1

VRF: VRF1
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

Gateway of last resort is not set

 C        192.168.10.0/24 is directly connected, Vlan10
 B I      192.168.20.1/32 [200/0] via VTEP 10.0.0.2 VNI 100999 router-mac 50:00:00:15:f4:e8 local-interface Vxlan1
 B I      192.168.30.1/32 [200/0] via VTEP 10.0.0.3 VNI 100999 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1
 B I      192.168.40.1/32 [200/0] via VTEP 10.0.0.3 VNI 100999 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1

```
PING:
```cfg
VPC1> ping 192.168.20.1

84 bytes from 192.168.20.1 icmp_seq=1 ttl=62 time=78.295 ms
84 bytes from 192.168.20.1 icmp_seq=2 ttl=62 time=82.967 ms
84 bytes from 192.168.20.1 icmp_seq=3 ttl=62 time=50.045 ms
84 bytes from 192.168.20.1 icmp_seq=4 ttl=62 time=50.310 ms
84 bytes from 192.168.20.1 icmp_seq=5 ttl=62 time=60.566 ms
^C
VPC1> ping 192.168.30.1

84 bytes from 192.168.30.1 icmp_seq=1 ttl=62 time=167.280 ms
84 bytes from 192.168.30.1 icmp_seq=2 ttl=62 time=42.138 ms
84 bytes from 192.168.30.1 icmp_seq=3 ttl=62 time=92.008 ms
84 bytes from 192.168.30.1 icmp_seq=4 ttl=62 time=68.540 ms
84 bytes from 192.168.30.1 icmp_seq=5 ttl=62 time=72.880 ms

VPC1> ping 192.168.40.1

84 bytes from 192.168.40.1 icmp_seq=1 ttl=62 time=69.257 ms
84 bytes from 192.168.40.1 icmp_seq=2 ttl=62 time=147.972 ms
84 bytes from 192.168.40.1 icmp_seq=3 ttl=62 time=102.205 ms
84 bytes from 192.168.40.1 icmp_seq=4 ttl=62 time=65.647 ms
84 bytes from 192.168.40.1 icmp_seq=5 ttl=62 time=41.620 ms

###########################################################

VPC3> ping 192.168.10.1

84 bytes from 192.168.10.1 icmp_seq=1 ttl=62 time=266.195 ms
84 bytes from 192.168.10.1 icmp_seq=2 ttl=62 time=96.733 ms
84 bytes from 192.168.10.1 icmp_seq=3 ttl=62 time=68.813 ms
84 bytes from 192.168.10.1 icmp_seq=4 ttl=62 time=66.950 ms
84 bytes from 192.168.10.1 icmp_seq=5 ttl=62 time=74.613 ms

VPC3> ping 192.168.20.1

84 bytes from 192.168.20.1 icmp_seq=1 ttl=62 time=114.358 ms
84 bytes from 192.168.20.1 icmp_seq=2 ttl=62 time=45.993 ms
84 bytes from 192.168.20.1 icmp_seq=3 ttl=62 time=45.179 ms
84 bytes from 192.168.20.1 icmp_seq=4 ttl=62 time=163.809 ms
84 bytes from 192.168.20.1 icmp_seq=5 ttl=62 time=45.459 ms

VPC3> ping 192.168.30.1

192.168.30.1 icmp_seq=1 ttl=64 time=0.001 ms
192.168.30.1 icmp_seq=2 ttl=64 time=0.001 ms
192.168.30.1 icmp_seq=3 ttl=64 time=0.001 ms
192.168.30.1 icmp_seq=4 ttl=64 time=0.001 ms
192.168.30.1 icmp_seq=5 ttl=64 time=0.001 ms

VPC3> ping 192.168.40.1

84 bytes from 192.168.40.1 icmp_seq=1 ttl=63 time=60.572 ms
84 bytes from 192.168.40.1 icmp_seq=2 ttl=63 time=22.861 ms
84 bytes from 192.168.40.1 icmp_seq=3 ttl=63 time=23.381 ms
84 bytes from 192.168.40.1 icmp_seq=4 ttl=63 time=37.362 ms
84 bytes from 192.168.40.1 icmp_seq=5 ttl=63 time=32.482 ms
```

### Конфиги устройств:
- [Spine-1](configs/S1.txt)
- [Spine-2](configs/S2.txt)
- [Leaf-1](configs/L1.txt)
- [Leaf-2](configs/L2.txt)
- [Leaf-3](configs/L3.txt)
