# Построение Underlay сети(ISIS)

### Цели:
- ##### Настроить ISIS для Underlay сети.

### Описание/Пошаговая инструкция выполнения домашнего задания:
- ##### Настроите ISIS в Underlay сети, для IP связанности между всеми сетевыми устройствами.
- ##### Зафиксируете в документации - план работы, адресное пространство, схему сети, конфигурацию устройств
- ##### Убедитесь в наличии IP связанности между устройствами в ISIS домене

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
| Spine-2   | 2001:db8:fab:ffff::2/128 |
| Leaf-1	| 2001:db8:fab:ffff::101/128 |
| Leaf-2	| 2001:db8:fab:ffff::102/128 |
| Leaf-3	| 2001:db8:fab:ffff::103/128 |

##### Общая таблица связей:
| Связь         | Подсеть                     | Адрес на Spine           | Адрес на Leaf           |
|---------------|-----------------------------|--------------------------|-------------------------|
| Spine1 <-> Leaf1  | 2001:db8:acad:1001::/64    | 2001:db8:acad:1001::1   | 2001:db8:acad:1001::2  |
| Spine1 <-> Leaf2  | 2001:db8:acad:1002::/64    | 2001:db8:acad:1002::1   | 2001:db8:acad:1002::2  |
| Spine1 <-> Leaf3  | 2001:db8:acad:1003::/64    | 2001:db8:acad:1003::1   | 2001:db8:acad:1003::2  |
| Spine2 <-> Leaf1  | 2001:db8:acad:2001::/64    | 2001:db8:acad:2001::1   | 2001:db8:acad:2001::2  |
| Spine2 <-> Leaf2  | 2001:db8:acad:2002::/64    | 2001:db8:acad:2002::1   | 2001:db8:acad:2002::2  |
| Spine2 <-> Leaf3  | 2001:db8:acad:2003::/64    | 2001:db8:acad:2003::1   | 2001:db8:acad:2003::2  |

##### Пример настроек:

Spine-1:
```cfg
router isis UNDERLAY
   net 49.0001.0100.0000.2002.00
   !
   address-family ipv4 unicast
   !
   address-family ipv6 unicast

int e1
   isis enable UNDERLAY
   isis bfd
   isis circuit-type level-1
   isis network point-to-point
   
int e2
   isis enable UNDERLAY
   isis bfd
   isis circuit-type level-1
   isis network point-to-point
   
int e3
   isis enable UNDERLAY
   isis bfd
   isis circuit-type level-1
   isis network point-to-point

int lo0
   isis enable UNDERLAY
```

Leaf-1:
```cfg
router isis UNDERLAY
   net 49.0001.0100.0000.0003.00
   !
   address-family ipv4 unicast
   !
   address-family ipv6 unicast

int e1
   isis enable UNDERLAY
   isis bfd
   isis circuit-type level-1
   isis network point-to-point
   
int e2
   isis enable UNDERLAY
   isis bfd
   isis circuit-type level-1
   isis network point-to-point
   
int lo0
   isis enable UNDERLAY
```

#### Проверка

##### IPv4

RIB:
```cfg
Leaf-3#show rib route ipv6
VRF: default, Protocol: connected
Codes: C - Connected, S - Static, P - Route Input
       B - BGP, O - OSPF, O3 - OSPF3, I - IS-IS
       > - Best Route, * - Unresolved Nexthop
       L - Part of a recursive route resolution loop
>C    2001:db8:fab:1003::/64 [0/1]
         via 2001:db8:fab:1003::2, Ethernet1
>C    2001:db8:fab:2003::/64 [0/1]
         via 2001:db8:fab:2003::2, Ethernet2
VRF: default, Protocol: ospf3
Codes: C - Connected, S - Static, P - Route Input
       B - BGP, O - OSPF, O3 - OSPF3, I - IS-IS
       > - Best Route, * - Unresolved Nexthop
       L - Part of a recursive route resolution loop
>O3    2001:db8:fab:1001::/64 [110/20]
         via fe80::5200:ff:fed5:5dc0, Ethernet1
>O3    2001:db8:fab:1002::/64 [110/20]
         via fe80::5200:ff:fed5:5dc0, Ethernet1
 O3    2001:db8:fab:1003::/64 [110/10]
         via 2001:db8:fab:1003::2, Ethernet1
>O3    2001:db8:fab:2001::/64 [110/20]
         via fe80::5200:ff:fe03:3766, Ethernet2
>O3    2001:db8:fab:2002::/64 [110/20]
         via fe80::5200:ff:fe03:3766, Ethernet2
 O3    2001:db8:fab:2003::/64 [110/10]
         via 2001:db8:fab:2003::2, Ethernet2
>O3    2001:db8:fab:ffff::1/128 [110/10]
         via fe80::5200:ff:fed5:5dc0, Ethernet1
>O3    2001:db8:fab:ffff::2/128 [110/10]
         via fe80::5200:ff:fe03:3766, Ethernet2
>O3    2001:db8:fab:ffff::101/128 [110/20]
         via fe80::5200:ff:fe03:3766, Ethernet2
>O3    2001:db8:fab:ffff::102/128 [110/20]
         via fe80::5200:ff:fed5:5dc0, Ethernet1
         via fe80::5200:ff:fe03:3766, Ethernet2
 O3    2001:db8:fab:ffff::103/128 [110/0]
         via 2001:db8:fab:ffff::103, Loopback0
VRF: default, Protocol: isis
Codes: C - Connected, S - Static, P - Route Input
       B - BGP, O - OSPF, O3 - OSPF3, I - IS-IS
       > - Best Route, * - Unresolved Nexthop
       L - Part of a recursive route resolution loop
 I    2001:db8:fab:1001::/64 [115/20]
         via fe80::5200:ff:fed5:5dc0, Ethernet1
 I    2001:db8:fab:1002::/64 [115/20]
         via fe80::5200:ff:fed5:5dc0, Ethernet1
 I    2001:db8:fab:1003::/64 [-115/10]
         via Null0, directly connected
 I    2001:db8:fab:2001::/64 [115/20]
         via fe80::5200:ff:fe03:3766, Ethernet2
 I    2001:db8:fab:2002::/64 [115/20]
         via fe80::5200:ff:fe03:3766, Ethernet2
 I    2001:db8:fab:2003::/64 [-115/10]
         via Null0, directly connected
 I    2001:db8:fab:ffff::1/128 [115/20]
         via fe80::5200:ff:fed5:5dc0, Ethernet1
 I    2001:db8:fab:ffff::2/128 [115/20]
         via fe80::5200:ff:fe03:3766, Ethernet2
 I    2001:db8:fab:ffff::101/128 [115/30]
         via fe80::5200:ff:fed5:5dc0, Ethernet1
         via fe80::5200:ff:fe03:3766, Ethernet2
 I    2001:db8:fab:ffff::102/128 [115/30]
         via fe80::5200:ff:fed5:5dc0, Ethernet1
         via fe80::5200:ff:fe03:3766, Ethernet2
 I    2001:db8:fab:ffff::103/128 [-115/10]
         via Null0, directly connected
```

FIB после отключения на маршртизаторах OSPF:
```cfg
Spine-1#sh ip ro

VRF: default
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

 I L1     10.0.0.1/32 [115/20] via 172.16.1.2, Ethernet1
 I L1     10.0.0.2/32 [115/20] via 172.16.2.2, Ethernet2
 I L1     10.0.0.3/32 [115/20] via 172.16.3.2, Ethernet3
 C        10.0.1.1/32 is directly connected, Loopback0
 I L1     10.0.2.2/32 [115/30] via 172.16.1.2, Ethernet1
                               via 172.16.2.2, Ethernet2
                               via 172.16.3.2, Ethernet3
 C        172.16.1.0/30 is directly connected, Ethernet1
 I L1     172.16.1.4/30 [115/20] via 172.16.1.2, Ethernet1
 C        172.16.2.0/30 is directly connected, Ethernet2
 I L1     172.16.2.4/30 [115/20] via 172.16.2.2, Ethernet2
 C        172.16.3.0/30 is directly connected, Ethernet3
 I L1     172.16.3.4/30 [115/20] via 172.16.3.2, Ethernet3
```

PING:
```cfg
Spine-1#ping 10.0.0.1
PING 10.0.0.1 (10.0.0.1) 72(100) bytes of data.
80 bytes from 10.0.0.1: icmp_seq=1 ttl=64 time=10.9 ms
80 bytes from 10.0.0.1: icmp_seq=2 ttl=64 time=13.5 ms
80 bytes from 10.0.0.1: icmp_seq=3 ttl=64 time=8.88 ms
80 bytes from 10.0.0.1: icmp_seq=4 ttl=64 time=21.2 ms
80 bytes from 10.0.0.1: icmp_seq=5 ttl=64 time=15.3 ms

--- 10.0.0.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 54ms
rtt min/avg/max/mdev = 8.886/13.985/21.264/4.250 ms, pipe 2, ipg/ewma 13.559/12.625 ms
Spine-1#ping 10.0.0.2
PING 10.0.0.2 (10.0.0.2) 72(100) bytes of data.
80 bytes from 10.0.0.2: icmp_seq=1 ttl=64 time=10.5 ms
80 bytes from 10.0.0.2: icmp_seq=2 ttl=64 time=22.6 ms
80 bytes from 10.0.0.2: icmp_seq=3 ttl=64 time=17.0 ms
80 bytes from 10.0.0.2: icmp_seq=4 ttl=64 time=15.7 ms
80 bytes from 10.0.0.2: icmp_seq=5 ttl=64 time=15.2 ms

--- 10.0.0.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 60ms
rtt min/avg/max/mdev = 10.533/16.266/22.688/3.902 ms, pipe 2, ipg/ewma 15.174/13.341 ms
Spine-1#ping 10.0.0.3
PING 10.0.0.3 (10.0.0.3) 72(100) bytes of data.
80 bytes from 10.0.0.3: icmp_seq=1 ttl=64 time=10.9 ms
80 bytes from 10.0.0.3: icmp_seq=2 ttl=64 time=21.6 ms
80 bytes from 10.0.0.3: icmp_seq=3 ttl=64 time=18.7 ms

--- 10.0.0.3 ping statistics ---
5 packets transmitted, 3 received, 40% packet loss, time 58ms
rtt min/avg/max/mdev = 10.978/17.120/21.604/4.495 ms, pipe 2, ipg/ewma 14.654/13.115 ms
Spine-1#ping 10.0.2.2
PING 10.0.2.2 (10.0.2.2) 72(100) bytes of data.
80 bytes from 10.0.2.2: icmp_seq=1 ttl=63 time=196 ms
80 bytes from 10.0.2.2: icmp_seq=2 ttl=63 time=205 ms
80 bytes from 10.0.2.2: icmp_seq=3 ttl=63 time=213 ms
80 bytes from 10.0.2.2: icmp_seq=4 ttl=63 time=209 ms
80 bytes from 10.0.2.2: icmp_seq=5 ttl=63 time=211 ms

--- 10.0.2.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 46ms
rtt min/avg/max/mdev = 196.794/207.345/213.885/6.004 ms, pipe 5, ipg/ewma 11.519/202.342 ms

```

##### IPv6
SH IPv6 RO
```cfg
Spine-1#SH IPv6 RO

VRF: default
Displaying 11 of 18 IPv6 routing table entries
Codes: C - connected, S - static, K - kernel, O3 - OSPFv3,
       B - Other BGP Routes, A B - BGP Aggregate, R - RIP,
       I L1 - IS-IS level 1, I L2 - IS-IS level 2, DH - DHCP,
       NG - Nexthop Group Static Route, M - Martian,
       DP - Dynamic Policy Route, L - VRF Leaked,
       RC - Route Cache Route

 C        2001:db8:fab:1001::/64 [0/1]
           via Ethernet1, directly connected
 C        2001:db8:fab:1002::/64 [0/1]
           via Ethernet2, directly connected
 C        2001:db8:fab:1003::/64 [0/1]
           via Ethernet3, directly connected
 I L1     2001:db8:fab:2001::/64 [115/20]
           via fe80::5200:ff:fecb:38c2, Ethernet1
 I L1     2001:db8:fab:2002::/64 [115/20]
           via fe80::5200:ff:fe15:f4e8, Ethernet2
 I L1     2001:db8:fab:2003::/64 [115/20]
           via fe80::5200:ff:fed7:ee0b, Ethernet3
 C        2001:db8:fab:ffff::1/128 [0/0]
           via Loopback0, directly connected
 I L1     2001:db8:fab:ffff::2/128 [115/30]
           via fe80::5200:ff:fecb:38c2, Ethernet1
           via fe80::5200:ff:fe15:f4e8, Ethernet2
           via fe80::5200:ff:fed7:ee0b, Ethernet3
 I L1     2001:db8:fab:ffff::101/128 [115/20]
           via fe80::5200:ff:fecb:38c2, Ethernet1
 I L1     2001:db8:fab:ffff::102/128 [115/20]
           via fe80::5200:ff:fe15:f4e8, Ethernet2
 I L1     2001:db8:fab:ffff::103/128 [115/20]
           via fe80::5200:ff:fed7:ee0b, Ethernet3
```

PING
```cfg
Spine-1#ping 2001:db8:fab:ffff::2
PING 2001:db8:fab:ffff::2(2001:db8:fab:ffff::2) 52 data bytes
60 bytes from 2001:db8:fab:ffff::2: icmp_seq=1 ttl=63 time=28.5 ms
60 bytes from 2001:db8:fab:ffff::2: icmp_seq=2 ttl=63 time=39.5 ms
60 bytes from 2001:db8:fab:ffff::2: icmp_seq=3 ttl=63 time=35.8 ms
60 bytes from 2001:db8:fab:ffff::2: icmp_seq=4 ttl=63 time=26.6 ms
60 bytes from 2001:db8:fab:ffff::2: icmp_seq=5 ttl=63 time=23.8 ms

--- 2001:db8:fab:ffff::2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 89ms
rtt min/avg/max/mdev = 23.862/30.898/39.519/5.859 ms, pipe 3, ipg/ewma 22.417/29.387 ms
Spine-1#2001:db8:fab:ffff::101
% Invalid input
Spine-1#ping 2001:db8:fab:ffff::2
PING 2001:db8:fab:ffff::2(2001:db8:fab:ffff::2) 52 data bytes
60 bytes from 2001:db8:fab:ffff::2: icmp_seq=1 ttl=63 time=28.5 ms
60 bytes from 2001:db8:fab:ffff::2: icmp_seq=2 ttl=63 time=33.1 ms
60 bytes from 2001:db8:fab:ffff::2: icmp_seq=3 ttl=63 time=32.8 ms
60 bytes from 2001:db8:fab:ffff::2: icmp_seq=4 ttl=63 time=32.4 ms
60 bytes from 2001:db8:fab:ffff::2: icmp_seq=5 ttl=63 time=41.2 ms

--- 2001:db8:fab:ffff::2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 86ms
rtt min/avg/max/mdev = 28.533/33.671/41.299/4.169 ms, pipe 3, ipg/ewma 21.668/31.365 ms
Spine-1#ping 2001:db8:fab:ffff::101
PING 2001:db8:fab:ffff::101(2001:db8:fab:ffff::101) 52 data bytes
60 bytes from 2001:db8:fab:ffff::101: icmp_seq=1 ttl=64 time=8.72 ms
60 bytes from 2001:db8:fab:ffff::101: icmp_seq=2 ttl=64 time=22.1 ms
60 bytes from 2001:db8:fab:ffff::101: icmp_seq=3 ttl=64 time=19.5 ms
60 bytes from 2001:db8:fab:ffff::101: icmp_seq=4 ttl=64 time=22.8 ms
60 bytes from 2001:db8:fab:ffff::101: icmp_seq=5 ttl=64 time=16.8 ms

--- 2001:db8:fab:ffff::101 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 55ms
rtt min/avg/max/mdev = 8.723/18.020/22.802/5.105 ms, pipe 3, ipg/ewma 13.777/13.442 ms
Spine-1#ping 2001:db8:fab:ffff::102
PING 2001:db8:fab:ffff::102(2001:db8:fab:ffff::102) 52 data bytes
60 bytes from 2001:db8:fab:ffff::102: icmp_seq=1 ttl=64 time=11.8 ms
60 bytes from 2001:db8:fab:ffff::102: icmp_seq=2 ttl=64 time=15.9 ms
60 bytes from 2001:db8:fab:ffff::102: icmp_seq=3 ttl=64 time=16.4 ms
60 bytes from 2001:db8:fab:ffff::102: icmp_seq=4 ttl=64 time=19.4 ms
60 bytes from 2001:db8:fab:ffff::102: icmp_seq=5 ttl=64 time=15.7 ms

--- 2001:db8:fab:ffff::102 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 67ms
rtt min/avg/max/mdev = 11.882/15.886/19.414/2.400 ms, pipe 2, ipg/ewma 16.979/13.966 ms
Spine-1#ping 2001:db8:fab:ffff::103
PING 2001:db8:fab:ffff::103(2001:db8:fab:ffff::103) 52 data bytes
60 bytes from 2001:db8:fab:ffff::103: icmp_seq=1 ttl=64 time=13.7 ms
60 bytes from 2001:db8:fab:ffff::103: icmp_seq=2 ttl=64 time=13.4 ms
60 bytes from 2001:db8:fab:ffff::103: icmp_seq=3 ttl=64 time=8.14 ms
60 bytes from 2001:db8:fab:ffff::103: icmp_seq=4 ttl=64 time=8.92 ms
60 bytes from 2001:db8:fab:ffff::103: icmp_seq=5 ttl=64 time=10.7 ms

--- 2001:db8:fab:ffff::103 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 65ms
rtt min/avg/max/mdev = 8.143/11.013/13.795/2.304 ms, ipg/ewma 16.445/12.310 ms
```

### Конфиги устройств:
- [Spine-1](configs/S1.txt)
- [Spine-2](configs/S2.txt)
- [Leaf-1](configs/L1.txt)
- [Leaf-2](configs/L2.txt)
- [Leaf-3](configs/L3.txt)
