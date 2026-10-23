&nbsp;**1. CORE-1 (ISR 4331)**

```
enable
configure terminal
hostname CORE-1
interface GigabitEthernet0/0/2
 ip address 10.49.0.1 255.255.255.252
 no shutdown
exit
interface GigabitEthernet0/0/1
 ip address 10.49.0.5 255.255.255.252
 no shutdown
exit
interface Serial0/1/0
 ip address 10.49.0.25 255.255.255.252
 encapsulation ppp
 clock rate 64000
 no shutdown
exit
router ospf 1
 router-id 1.1.1.1
 network 10.49.0.0 0.0.0.3 area 0
 network 10.49.0.4 0.0.0.3 area 0
 network 10.49.0.24 0.0.0.3 area 0
 redistribute static subnets
exit
ip route 10.49.80.0 255.255.255.0 10.49.0.26
ip route 10.49.16.0 255.255.255.0 10.49.0.6
end
write memory
```

**2\. CORE-2 (ISR 4331)**

```
enable
configure terminal
hostname CORE-2
interface GigabitEthernet0/0/2
 ip address 10.49.0.2 255.255.255.252
 no shutdown
exit
interface GigabitEthernet0/0/1
 ip address 10.49.0.9 255.255.255.252
 no shutdown
exit
router ospf 1
 router-id 2.2.2.2
 network 10.49.0.0 0.0.0.3 area 0
 network 10.49.0.8 0.0.0.3 area 0
 redistribute eigrp 10 subnets
exit
router eigrp 10
 network 10.49.0.8 0.0.0.3
 redistribute ospf 1 metric 10000 100 255 1 1500
 no auto-summary
exit
ip route 10.49.32.0 255.255.255.0 10.49.0.10
ip route 10.49.48.0 255.255.255.0 10.49.0.14
end
write memory
```

**3\. CORE-3 (ISR 4331)**

```
enable
configure terminal
hostname CORE-3
interface GigabitEthernet0/0/0
 ip address 10.49.0.6 255.255.255.252
 no shutdown
exit
interface GigabitEthernet0/0/1
 ip address 10.49.0.10 255.255.255.252
 no shutdown
exit
router ospf 1
 router-id 3.3.3.3
 network 10.49.0.4 0.0.0.3 area 0
 network 10.49.0.8 0.0.0.3 area 0
exit
end
write memory
```

**4\. PUENTE (ISR 2911)**

```
enable
configure terminal
hostname PUENTE
interface Serial0/1/0
 ip address 10.49.0.10 255.255.255.252
 encapsulation ppp
 clock rate 64000
 no shutdown
exit
interface GigabitEthernet0/0/0
 ip address 10.49.0.13 255.255.255.252
 no shutdown
exit
interface GigabitEthernet0/0/1
 ip address 10.49.0.17 255.255.255.252
 no shutdown
exit
router ospf 1
 network 10.49.0.8 0.0.0.3 area 0
 network 10.49.0.12 0.0.0.3 area 0
 network 10.49.0.16 0.0.0.3 area 0
exit
ip route 0.0.0.0 0.0.0.0 10.49.0.9
end
write memory
```

**5\. SW-OCCIDENTE (Multicapa 3560)**

```
enable
configure terminal
hostname SW-OCCIDENTE
ip routing
interface GigabitEthernet0/1
 no switchport
 ip address 10.49.0.6 255.255.255.252
 no shutdown
exit
interface GigabitEthernet0/2
 no switchport
 ip address 10.49.0.14 255.255.255.252
 no shutdown
exit
router ospf 1
 network 10.49.0.4 0.0.0.3 area 0
 network 10.49.0.12 0.0.0.3 area 0
exit
ip route 0.0.0.0 0.0.0.0 10.49.0.5
ip route 10.49.16.0 255.255.255.0 10.49.0.13
end
write memory
```

**6\. SW-CORE-3 (Multicapa 3560)**

```
enable
configure terminal
hostname SW-CORE-3
ip routing
interface GigabitEthernet0/1
 no switchport
 ip address 10.49.0.10 255.255.255.252
 no shutdown
exit
interface FastEthernet0/1
 no switchport
 ip address 10.49.0.13 255.255.255.252
 no shutdown
exit
interface FastEthernet0/2
 no switchport
 ip address 10.49.0.17 255.255.255.252
 no shutdown
exit
router ospf 1
 network 10.49.0.8 0.0.0.3 area 0
 network 10.49.0.12 0.0.0.3 area 0
 network 10.49.0.16 0.0.0.3 area 0
exit
ip route 0.0.0.0 0.0.0.0 10.49.0.9
ip route 10.49.32.0 255.255.255.0 10.49.0.14
ip route 10.49.48.0 255.255.255.0 10.49.0.18
end
write memory
```

**7\. R-OCCIDENTE (ISR 2911)**

```
enable
configure terminal
hostname R-OCCIDENTE
interface GigabitEthernet0/0
 ip address 10.49.0.13 255.255.255.252
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
 network 10.49.0.12 0.0.0.3 area 0
 network 10.49.16.0 0.0.0.255 area 0
exit
ip route 0.0.0.0 0.0.0.0 10.49.0.14
end
write memory
```

**8\. R-NORTE (ISR 2911)**

```
enable
configure terminal
hostname R-NORTE
interface GigabitEthernet0/0
 ip address 10.49.0.14 255.255.255.252
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
 network 10.49.0.12 0.0.0.3
 network 10.49.32.0 0.0.0.255
 no auto-summary
exit
ip route 0.0.0.0 0.0.0.0 10.49.0.13
end
write memory
```

**9\. R-ORIENTE (ISR 2911)**

```
enable
configure terminal
hostname R-ORIENTE
interface GigabitEthernet0/0
 ip address 10.49.0.18 255.255.255.252
 no shutdown
exit
interface GigabitEthernet0/1
 ip address 10.49.48.2 255.255.255.192
 no shutdown
exit
router rip
 version 2
 network 10.49.0.16
 network 10.49.48.0
 no auto-summary
exit
ip route 0.0.0.0 0.0.0.0 10.49.0.17
end
write memory
```

**10\. R-CENTRAL1 (ISR 4331)**

```
enable
configure terminal
hostname R-CENTRAL1
interface Serial0/1/0
 ip address 10.49.0.26 255.255.255.252
 encapsulation ppp
 no shutdown
exit
interface GigabitEthernet0/0/1.23
 encapsulation dot1Q 23
 ip address 10.49.80.1 255.255.255.0
 no shutdown
exit
interface GigabitEthernet0/0/1
 no ip address
 no shutdown
exit
router ospf 1
 network 10.49.0.24 0.0.0.3 area 0
 redistribute static subnets
exit
ip route 10.49.0.0 255.255.0.0 10.49.0.25
end
write memory
```

**11\. R-CENTRAL2 (ISR 4331)**

```
enable
configure terminal
hostname R-CENTRAL2
interface GigabitEthernet0/0/0
 ip address 10.49.64.2 255.255.255.252
 no shutdown
exit
interface GigabitEthernet0/0/1.13
 encapsulation dot1Q 13
 ip address 10.49.70.1 255.255.255.0
exit
interface GigabitEthernet0/0/1.23
 encapsulation dot1Q 23
 ip address 10.49.80.1 255.255.255.0
exit
interface GigabitEthernet0/0/1.33
 encapsulation dot1Q 33
 ip address 10.49.90.1 255.255.255.0
exit
ip route 0.0.0.0 0.0.0.0 10.49.64.1
end
write memory
```

**12\. SWITCHES DE ACCESO Y VTP**

**SW-OCC-1:**

```
enable
configure terminal
hostname SW-OCC-1
vtp domain bantech49
vtp password 201407049
vtp mode server
vlan 453
 name Cajas
vlan 304
 name Asesores
vlan 105
 name Gerencia
vlan 126
 name Seguridad
exit
interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk encapsulation dot1q
 no shutdown
exit
interface range FastEthernet0/1-2
 switchport mode access
 switchport access vlan 453
 no shutdown
exit
interface range FastEthernet0/3-4
 switchport mode access
 switchport access vlan 304
 no shutdown
exit
interface FastEthernet0/5
 switchport mode access
 switchport access vlan 105
 no shutdown
exit
interface range FastEthernet0/6-7
 switchport mode access
 switchport access vlan 126
 no shutdown
exit
spanning-tree vlan 453,304,105,126 root primary
end
write memory
```

**SW-NORTE-1:**

```
enable
configure terminal
hostname SW-NORTE-1
vtp domain bantech49
vtp password 201407049
vtp mode server
vlan 507
 name Analisis
vlan 258
 name Auditoria
vlan 149
 name Legal
exit
spanning-tree mode rapid-pvst
spanning-tree vlan 507,258,149 root primary
interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk encapsulation dot1q
 no shutdown
exit
interface GigabitEthernet0/2
 switchport mode trunk
 switchport trunk encapsulation dot1q
 no shutdown
exit
interface FastEthernet0/1
 switchport mode trunk
 switchport trunk encapsulation dot1q
 no shutdown
exit
interface range FastEthernet0/2-15
 switchport mode access
 switchport access vlan 507
 no shutdown
exit
interface range FastEthernet0/16-20
 switchport mode access
 switchport access vlan 258
 no shutdown
exit
interface range FastEthernet0/21-24
 switchport mode access
 switchport access vlan 149
 no shutdown
exit
end
write memory
```

**SW-NORTE-2:**

```
enable
configure terminal
hostname SW-NORTE-2
vtp domain bantech49
vtp password 201407049
vtp mode client
spanning-tree mode rapid-pvst
interface range GigabitEthernet0/1-2
 switchport mode trunk
 switchport trunk encapsulation dot1q
 no shutdown
exit
interface range FastEthernet0/1-10
 switchport mode access
 switchport access vlan 507
 no shutdown
exit
interface range FastEthernet0/11-15
 switchport mode access
 switchport access vlan 258
 no shutdown
exit
interface range FastEthernet0/16-24
 switchport mode access
 switchport access vlan 149
 no shutdown
exit
end
write memory
```

**SW-NORTE-3:**

```
enable
configure terminal
hostname SW-NORTE-3
vtp domain bantech49
vtp password 201407049
vtp mode client
spanning-tree mode rapid-pvst
interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk encapsulation dot1q
 no shutdown
exit
interface FastEthernet0/1
 switchport mode trunk
 switchport trunk encapsulation dot1q
 no shutdown
exit
interface range FastEthernet0/2-10
 switchport mode access
 switchport access vlan 507
 no shutdown
exit
interface range FastEthernet0/11-15
 switchport mode access
 switchport access vlan 258
 no shutdown
exit
interface range FastEthernet0/16-24
 switchport mode access
 switchport access vlan 149
 no shutdown
exit
end
write memory
```

**MS1 (Sede Oriente):**

```
enable
configure terminal
hostname MS1
ip routing
vtp domain bantech49
vtp password 201407049
vtp mode server
vlan 410
 name Boveda
vlan 611
 name Plataforma
exit
interface range GigabitEthernet0/1-2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 no shutdown
exit
interface FastEthernet0/24
 switchport trunk encapsulation dot1q
 switchport mode trunk
 no shutdown
exit
interface Vlan410
 ip address 10.49.48.3 255.255.255.192
 standby 1 ip 10.49.48.1
 standby 1 priority 150
 standby 1 preempt
 no shutdown
exit
interface Vlan611
 ip address 10.49.48.67 255.255.255.192
 standby 2 ip 10.49.48.65
 standby 2 priority 150
 standby 2 preempt
 no shutdown
exit
spanning-tree vlan 410,611 root primary
end
write memory
```

**MS2 (Sede Oriente):**

```
enable
configure terminal
hostname MS2
ip routing
vtp domain bantech49
vtp password 201407049
vtp mode server
vlan 410
 name Boveda
vlan 611
 name Plataforma
exit
interface GigabitEthernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 no shutdown
exit
interface FastEthernet0/24
 switchport trunk encapsulation dot1q
 switchport mode trunk
 no shutdown
exit
interface Vlan410
 ip address 10.49.48.4 255.255.255.192
 standby 1 ip 10.49.48.1
 standby 1 priority 100
 standby 1 preempt
 no shutdown
exit
interface Vlan611
 ip address 10.49.48.68 255.255.255.192
 standby 2 ip 10.49.48.65
 standby 2 priority 100
 standby 2 preempt
 no shutdown
exit
spanning-tree vlan 410,611 root secondary
end
write memory
```

**SW-ORIENTE (Switch de acceso):**

```
enable
configure terminal
hostname SW-ORIENTE
vtp domain bantech49
vtp password 201407049
vtp mode client
interface range GigabitEthernet0/1-2
 switchport mode trunk
 switchport trunk encapsulation dot1q
 no shutdown
exit
interface range FastEthernet0/1-12
 switchport mode access
 switchport access vlan 410
 no shutdown
exit
interface range FastEthernet0/13-24
 switchport mode access
 switchport access vlan 611
 no shutdown
exit
end
write memory
```

**SW-DC-DIST:**

```
enable
configure terminal
hostname SW-DC-DIST
vtp domain bantech49
vtp password 201407049
vtp mode server
vlan 13
 name Core_BD
vlan 23
 name Web_Apps
vlan 33
 name NOC
exit
interface GigabitEthernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 no shutdown
exit
interface range FastEthernet0/23-24
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 1 mode active
 no shutdown
exit
interface Port-channel1
 switchport trunk encapsulation dot1q
 switchport mode trunk
exit
interface FastEthernet0/4
 switchport mode access
 switchport access vlan 23
 no shutdown
exit
interface range FastEthernet0/1-2
 switchport mode access
 switchport access vlan 13
 no shutdown
exit
end
write memory
```

**SW-DC-ACC:**

```
enable
configure terminal
hostname SW-DC-ACC
vtp domain bantech49
vtp password 201407049
vtp mode client
interface range GigabitEthernet0/1-2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 1 mode active
 no shutdown
exit
interface Port-channel1
 switchport trunk encapsulation dot1q
 switchport mode trunk
exit
interface FastEthernet0/24
 switchport mode trunk
 switchport trunk encapsulation dot1q
 no shutdown
exit
interface FastEthernet0/3
 switchport mode access
 switchport access vlan 23
 no shutdown
exit
end
write memory
```

**13\. DIRECCIONAMIENTO IP DE PCS Y SERVIDORES**

_(Ingresa a cada dispositivo > Desktop > IP Configuration)_

**Occidente:**

- **PC0:** IP 10.49.16.2, Mask 255.255.255.192, GW 10.49.16.1
- **PC1:** IP 10.49.16.3, Mask 255.255.255.192, GW 10.49.16.1
- **PC3 (Asesores):** IP 10.49.16.66, Mask 255.255.255.224, GW 10.49.16.65

**Norte:**

- **PC8:** IP 10.49.32.2, Mask 255.255.255.192, GW 10.49.32.1
- **PC9:** IP 10.49.32.3, Mask 255.255.255.192, GW 10.49.32.1
- **PC10:** IP 10.49.32.4, Mask 255.255.255.192, GW 10.49.32.1
- **PC11:** IP 10.49.32.5, Mask 255.255.255.192, GW 10.49.32.1

**Oriente:**

- **PC2:** IP 10.49.48.5, Mask 255.255.255.192, GW 10.49.48.1
- **PC15:** IP 10.49.48.6, Mask 255.255.255.192, GW 10.49.48.1
- **PC16:** IP 10.49.48.7, Mask 255.255.255.192, GW 10.49.48.1
- **PC17:** IP 10.49.48.69, Mask 255.255.255.192, GW 10.49.48.65

**Data Center:**

- **SRV-WEB-2:** IP 10.49.80.19, Mask 255.255.255.0, GW 10.49.80.1
- **SRV-DB-1:** IP 10.49.70.2, Mask 255.255.255.0, GW 10.49.70.1
- **SRV-NOC-1:** IP 10.49.90.2, Mask 255.255.255.0, GW 10.49.90.1