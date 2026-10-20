**1\. CORE-1 (ISR 4331)**

```
enable
configure terminal
hostname CORE-1
! Fibra hacia CORE-2
interface GigabitEthernet0/0/0
 ip address 10.49.0.1 255.255.255.252
 no shutdown
exit
! Enlace hacia CORE-3
interface GigabitEthernet0/0/1
 ip address 10.49.0.5 255.255.255.252
 no shutdown
exit
! Enlace Serial WAN hacia R-CENTRAL1
interface Serial0/1/0
 ip address 10.49.0.29 255.255.255.252
 encapsulation ppp
 clock rate 64000
 no shutdown
exit
! OSPF y Redistribución
router ospf 1
 router-id 1.1.1.1
 network 10.49.0.0 0.0.0.3 area 0
 network 10.49.0.4 0.0.0.3 area 0
 network 10.49.0.28 0.0.0.3 area 0
 redistribute static subnets
 passive-interface default
 no passive-interface GigabitEthernet0/0/1
 no passive-interface Serial0/1/0
exit
ip route 10.49.64.0 255.255.255.192 10.49.0.30
end
write memory
```

**2\. CORE-2 (ISR 4331)**

```
enable
configure terminal
hostname CORE-2
! Fibra hacia CORE-1
interface GigabitEthernet0/0/0
 ip address 10.49.0.2 255.255.255.252
 no shutdown
exit
! Enlace hacia CORE-3
interface GigabitEthernet0/0/1
 ip address 10.49.0.9 255.255.255.252
 no shutdown
exit
! OSPF + EIGRP + Redistribución
router ospf 1
 router-id 2.2.2.2
 network 10.49.0.0 0.0.0.3 area 0
 network 10.49.0.8 0.0.0.3 area 0
 redistribute eigrp 10 subnets
 passive-interface default
 no passive-interface GigabitEthernet0/0/1
exit
router eigrp 10
 network 10.49.0.8 0.0.0.3
 redistribute ospf 1 metric 10000 100 255 1 1500
 no auto-summary
exit
end
write memory
```

**3\. CORE-3 (ISR 4331)**

```
enable
configure terminal
hostname CORE-3
! Enlaces al Núcleo
interface GigabitEthernet0/0/0
 ip address 10.49.0.6 255.255.255.252
 no shutdown
exit
interface GigabitEthernet0/0/1
 ip address 10.49.0.10 255.255.255.252
 no shutdown
exit
! Enlace directo a R-OCCIDENTE
interface GigabitEthernet0/0/2
 ip address 10.49.0.13 255.255.255.252
 no shutdown
exit
! Redistribución Total
router ospf 1
 router-id 3.3.3.3
 network 10.49.0.4 0.0.0.3 area 0
 network 10.49.0.8 0.0.0.3 area 0
 network 10.49.0.12 0.0.0.3 area 0
 network 10.49.0.16 0.0.0.3 area 0
 redistribute eigrp 10 subnets
 redistribute rip subnets
 redistribute static subnets
exit
router eigrp 10
 network 10.49.0.8 0.0.0.3
 network 10.49.0.20 0.0.0.3
 redistribute ospf 1 metric 10000 100 255 1 1500
 redistribute rip metric 10000 100 255 1 1500
 no auto-summary
exit
router rip
 version 2
 network 10.49.0.20
 redistribute ospf 1 metric 2
 redistribute eigrp 10 metric 2
 no auto-summary
exit
end
write memory
```

**4\. SW-CORE-3 (Switch Multicapa 3560)**

_Conectado a CORE-3 (Gi0/1 y Gi0/2), R-NORTE (Fa0/1) y R-ORIENTE (Fa0/2)._

```
enable
configure terminal
hostname SW-CORE-3
ip routing
! Enlaces hacia CORE-3
interface GigabitEthernet0/1
 no switchport
 ip address 10.49.0.17 255.255.255.252
 no shutdown
exit
interface GigabitEthernet0/2
 no switchport
 ip address 10.49.0.21 255.255.255.252
 no shutdown
exit
! Enlaces hacia R-NORTE y R-ORIENTE
interface FastEthernet0/1
 switchport mode access
 no shutdown
exit
interface FastEthernet0/2
 switchport mode access
 no shutdown
exit
! Protocolos
router ospf 1
 network 10.49.0.16 0.0.0.3 area 0
 network 10.49.0.20 0.0.0.3 area 0
exit
router eigrp 10
 network 10.49.0.16 0.0.0.3
 no auto-summary
exit
router rip
 version 2
 network 10.49.0.20
 no auto-summary
exit
ip route 0.0.0.0 0.0.0.0 10.49.0.6
end
write memory
```

**5\. R-CENTRAL1 (ISR 4331)**

```
enable
configure terminal
hostname R-CENTRAL1
interface Serial0/1/0
 ip address 10.49.0.30 255.255.255.252
 encapsulation ppp
 clock rate 64000
 no shutdown
exit
interface GigabitEthernet0/0/0
 ip address 10.49.64.1 255.255.255.252
 no shutdown
exit
interface GigabitEthernet0/0/1
 ip address 10.49.64.5 255.255.255.252
 no shutdown
exit
ip route 10.49.0.0 255.255.0.0 10.49.0.29
router ospf 1
 router-id 4.4.4.4
 network 10.49.0.28 0.0.0.3 area 0
 network 10.49.64.0 0.0.0.3 area 0
 redistribute static subnets
exit
end
write memory
```

**6\. R-OCCIDENTE (ISR 2911)**

```
enable
configure terminal
hostname R-OCCIDENTE
interface GigabitEthernet0/0
 ip address 10.49.0.14 255.255.255.252
 no shutdown
exit
interface GigabitEthernet0/1
 no shutdown
exit
interface GigabitEthernet0/1.453
 encapsulation dot1Q 453
 ip address 10.49.16.1 255.255.255.192
exit
interface GigabitEthernet0/1.304
 encapsulation dot1Q 304
 ip address 10.49.16.65 255.255.255.224
exit
interface GigabitEthernet0/1.105
 encapsulation dot1Q 105
 ip address 10.49.16.97 255.255.255.240
exit
interface GigabitEthernet0/1.126
 encapsulation dot1Q 126
 ip address 10.49.16.113 255.255.255.240
exit
router ospf 1
 router-id 5.5.5.5
 network 10.49.0.12 0.0.0.3 area 0
 network 10.49.16.0 0.0.0.255 area 0
 passive-interface GigabitEthernet0/1.453
 passive-interface GigabitEthernet0/1.304
 passive-interface GigabitEthernet0/1.105
 passive-interface GigabitEthernet0/1.126
exit
end
write memory
```

**7\. R-NORTE (ISR 2911)**

```
enable
configure terminal
hostname R-NORTE
interface GigabitEthernet0/0
 ip address 10.49.0.18 255.255.255.252
 no shutdown
exit
interface GigabitEthernet0/1
 no shutdown
exit
interface GigabitEthernet0/1.507
 encapsulation dot1Q 507
 ip address 10.49.32.1 255.255.255.192
exit
interface GigabitEthernet0/1.258
 encapsulation dot1Q 258
 ip address 10.49.32.65 255.255.255.224
exit
interface GigabitEthernet0/1.149
 encapsulation dot1Q 149
 ip address 10.49.32.97 255.255.255.240
exit
router eigrp 10
 network 10.49.0.16 0.0.0.3
 network 10.49.32.0 0.0.0.255
 no auto-summary
exit
end
write memory
```

**8\. R-ORIENTE (ISR 2911)**

```
enable
configure terminal
hostname R-ORIENTE
interface GigabitEthernet0/0
 ip address 10.49.0.22 255.255.255.252
 no shutdown
exit
interface GigabitEthernet0/1
 ip address 10.49.48.2 255.255.255.192
 no shutdown
exit
router rip
 version 2
 network 10.49.0.20
 network 10.49.48.0
 no auto-summary
exit
end
write memory
```


**9\. SW-OCC-1**
```

enable
configure terminal
hostname SW-OCC-1
spanning-tree mode rapid-pvst
vlan 12
 name Cajas
vlan 22
 name Asesores
vlan 32
 name Gerencia
vlan 42
 name Seguridad
 exit
interface GigabitEthernet0/1
 description Trunk hacia R-OCCIDENTE
 switchport mode trunk
 exit
interface range FastEthernet0/1 - 2
 switchport mode access
 switchport access vlan 12
 spanning-tree portfast
 exit
interface range FastEthernet0/3 - 4
 switchport mode access
 switchport access vlan 22
 spanning-tree portfast
 exit
interface FastEthernet0/5
 switchport mode access
 switchport access vlan 32
 spanning-tree portfast
 exit
interface range FastEthernet0/6 - 7
 switchport mode access
 switchport access vlan 42
 spanning-tree portfast
 exit
end
write memory
```

**10\. SW-DC-DIST (3560)**
```
enable
configure terminal
hostname SW-DC-DIST
spanning-tree mode rapid-pvst
vtp domain bantech49
vtp password cisco
vtp mode server
vlan 13
 name Core_BD
vlan 23
 name Web_Apps
vlan 33
 name NOC
 exit
spanning-tree vlan 13,23,33 priority 4096
! EtherChannel hacia SW-DC-ACC (Gi0/1 y Gi0/2 del 2960)
interface range FastEthernet0/23 - 24
 description EtherChannel hacia SW-DC-ACC
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-protocol lacp
 channel-group 1 mode active
 no shutdown
 exit
interface Port-channel1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 exit
! Servidores
interface range FastEthernet0/1 - 2
 description Servidores DB
 switchport mode access
 switchport access vlan 13
 spanning-tree portfast
 exit
interface range FastEthernet0/3 - 4
 description Servidores WEB
 switchport mode access
 switchport access vlan 23
 spanning-tree portfast
 exit
! Cable sobrante hacia R-CENTRAL1
interface GigabitEthernet0/1
 description Sin uso (cable a R-CENTRAL1)
 shutdown
 exit
end
write memory

**11\. SW-DC-ACC (2960)**
```
enable
configure terminal
hostname SW-DC-ACC
spanning-tree mode rapid-pvst
vtp domain bantech49
vtp password cisco
vtp mode client
! EtherChannel hacia SW-DC-DIST (Fa0/23-24 del 3560)
interface range GigabitEthernet0/1 - 2
 description EtherChannel hacia SW-DC-DIST
 switchport mode trunk
 channel-protocol lacp
 channel-group 1 mode active
 no shutdown
 exit
interface Port-channel1
 switchport mode trunk
 exit
! Trunk hacia R-CENTRAL2 (Gi0/0/1)
interface FastEthernet0/24
 description Trunk hacia R-CENTRAL2
 switchport mode trunk
 no shutdown
 exit
! Puerto sin uso
interface FastEthernet0/23
 shutdown
 exit
end
show vlan brief
```
configure terminal
interface range FastEthernet0/1 - 2
 description Servidores NOC
 switchport mode access
 switchport access vlan 33
 spanning-tree portfast
 exit
interface FastEthernet0/3
 description Servidor WEB-3
 switchport mode access
 switchport access vlan 23
 spanning-tree portfast
 exit
end
write memory
```
**13\. R-CENTRAL2 (ISR 4331)**
```
enable
configure terminal
hostname R-CENTRAL2
interface GigabitEthernet0/0/0
 description Enlace a R-CENTRAL1
 ip address 10.49.64.2 255.255.255.252
 no shutdown
 exit
interface GigabitEthernet0/0/1
 description Trunk hacia SW-DC-ACC
 no ip address
 no shutdown
 exit
interface GigabitEthernet0/0/1.13
 description Core_BD
 encapsulation dot1Q 13
 ip address 10.49.64.129 255.255.255.224
 exit
interface GigabitEthernet0/0/1.23
 description Web_Apps
 encapsulation dot1Q 23
 ip address 10.49.64.161 255.255.255.224
 exit
interface GigabitEthernet0/0/1.33
 description NOC
 encapsulation dot1Q 33
 ip address 10.49.64.193 255.255.255.240
 exit
ip route 0.0.0.0 0.0.0.0 10.49.64.1
end
write memory