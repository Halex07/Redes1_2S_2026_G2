# Proyecto 2: Infraestructura Financiera y Alta Disponibilidad BanTech GT

**Curso:** Redes de Computadoras 1, Segundo Semestre 2026
**Universidad de San Carlos de Guatemala, Facultad de Ingeniería**
**Grupo:** 2
| **Estudiantes:** | 
| Henry Alexander García Montúfar  | **Carné** | 201407049 |
| Edy Donaldo López Anavizca | **Carné** | 199812349 |
| Pablo Isaí Matusalen Cutzal Mazariegos  | **Carné** | 202209622 |



| Parámetro | Valor | Origen |
|---|---|---|
| Carnet del coordinador | 201407049 | Dato del grupo |
| **XX** | **49** | Últimos dos dígitos del carnet |
| **Y** | **2** | Último dígito del número de grupo |
| Dominio VTP | `bantech49` | `bantechXX` |
| Password VTP | `cisco` | Enunciado |

> Este documento cubre el **Área 1 (Data Center / Sede Central)** y el **Área 2 (Sede Occidente)**. El backbone y las Áreas 3 y 4 (Norte y Oriente) se documentan en las secciones de sus responsables.

---

## Índice

1. [Resumen de la solución](#1-resumen-de-la-solución)
2. [Mapa de enrutamiento por sede](#2-mapa-de-enrutamiento-por-sede)
3. [Tabla de VLANs](#3-tabla-de-vlans)
4. [Tablas de subneteo e IPs asignadas](#4-tablas-de-subneteo-e-ips-asignadas)
5. [Área 1: Data Center y Sede Central](#5-área-1-data-center-y-sede-central)
6. [Área 2: Sede Occidente](#6-área-2-sede-occidente)
7. [Pruebas y evidencia](#7-pruebas-y-evidencia)
8. [Problemas encontrados y soluciones](#8-problemas-encontrados-y-soluciones)
9. [Estado actual y pendientes](#9-estado-actual-y-pendientes)
10. [Flujo de trabajo en Git](#10-flujo-de-trabajo-en-git)

---

## 1. Resumen de la solución

| Área | Sede | Dominio de enrutamiento | Inter-VLAN | Alta disponibilidad |
|---|---|---|---|---|
| 1 | Data Center / Sede Central | **Rutas estáticas** (redistribuidas en OSPF por R-CENTRAL1) | Router-on-a-Stick en R-CENTRAL2 | EtherChannel LACP (Po1) + Rapid PVST+ con root forzado |
| 2 | Occidente | **OSPF** | Router-on-a-Stick en R-OCCIDENTE | Segmentación por VLAN; un solo switch de acceso |

---

## 2. Mapa de enrutamiento por sede

| Sede | Protocolo | Equipo frontera | Justificación |
|---|---|---|---|
| Data Center / Central | Estático | R-CENTRAL2 (acceso) y R-CENTRAL1 (redistribución) | El enunciado exige aislar el segmento por seguridad. R-CENTRAL2 solo tiene rutas estáticas y R-CENTRAL1 las propaga al backbone sin exponer el Data Center directamente al núcleo. |
| Occidente | OSPF | R-OCCIDENTE | Es la agencia de mayor volumen transaccional. OSPF converge rápido y escala si se agregan VLANs o subredes. |

---

## 3. Tabla de VLANs

Con **Y = 2**:

| Sede | VLAN | ID | Hosts requeridos | Equipo donde se crea |
|---|---|---|---|---|
| Occidente | Cajas | 12 | 45 | SW-OCC-1 |
| Occidente | Asesores | 22 | 30 | SW-OCC-1 |
| Occidente | Gerencia | 32 | 10 | SW-OCC-1 |
| Occidente | Seguridad | 42 | 12 | SW-OCC-1 |
| Data Center | Core_BD | 13 | 14 | SW-DC-DIST (VTP Server) |
| Data Center | Web_Apps | 23 | 28 | SW-DC-DIST (VTP Server) |
| Data Center | NOC | 33 | 10 | SW-DC-DIST (VTP Server) |

Las VLANs del Data Center usan la fórmula `1Y+1`, `2Y+1` y `3Y+1` del enunciado, que con Y = 2 da 13, 23 y 33.

---

## 4. Tablas de subneteo e IPs asignadas

Las subredes se calculan con **VLSM**. Cada subred debe alojar los hosts requeridos más la IP del gateway.

### 4.1 Occidente (bloque 10.49.16.0/24)

| VLAN | Hosts | Prefijo | Máscara | Red | Gateway | Rango de hosts | Broadcast |
|---|---|---|---|---|---|---|---|
| Cajas (12) | 45 | /26 | 255.255.255.192 | 10.49.16.0 | 10.49.16.1 | .2 a .62 | 10.49.16.63 |
| Asesores (22) | 30 | /26 | 255.255.255.192 | 10.49.16.64 | 10.49.16.65 | .66 a .126 | 10.49.16.127 |
| Gerencia (32) | 10 | /28 | 255.255.255.240 | 10.49.16.128 | 10.49.16.129 | .130 a .142 | 10.49.16.143 |
| Seguridad (42) | 12 | /28 | 255.255.255.240 | 10.49.16.144 | 10.49.16.145 | .146 a .158 | 10.49.16.159 |

Asesores usa /26 porque en un /27 (30 hosts útiles) no cabe la IP del gateway.

**Dispositivos finales de Occidente (7 PCs):**

| Dispositivo | VLAN | Puerto SW-OCC-1 | IP | Máscara | Gateway |
|---|---|---|---|---|---|
| PC Cajas 1 | 12 | Fa0/1 | 10.49.16.2 | 255.255.255.192 | 10.49.16.1 |
| PC Cajas 2 | 12 | Fa0/2 | 10.49.16.3 | 255.255.255.192 | 10.49.16.1 |
| PC Asesores 1 | 22 | Fa0/3 | 10.49.16.66 | 255.255.255.192 | 10.49.16.65 |
| PC Asesores 2 | 22 | Fa0/4 | 10.49.16.67 | 255.255.255.192 | 10.49.16.65 |
| PC Gerencia | 32 | Fa0/5 | 10.49.16.130 | 255.255.255.240 | 10.49.16.129 |
| PC Seguridad 1 | 42 | Fa0/6 | 10.49.16.146 | 255.255.255.240 | 10.49.16.145 |
| PC Seguridad 2 | 42 | Fa0/7 | 10.49.16.147 | 255.255.255.240 | 10.49.16.145 |

**Enlace de R-OCCIDENTE hacia el backbone:** 10.49.0.36/30, con R-OCCIDENTE Gi0/0 en 10.49.0.38 y el equipo vecino en 10.49.0.37.

### 4.2 Data Center (bloque 10.49.64.128/25 más enlace 10.49.64.0/30)

El bloque 10.49.64.0/26 que traía el backbone no alcanza para las tres VLANs, por lo que se reservó 10.49.64.128/25.

| Red | Hosts | Prefijo | Máscara | Red | Gateway | Rango de hosts | Broadcast |
|---|---|---|---|---|---|---|---|
| Core_BD (13) | 14 | /27 | 255.255.255.224 | 10.49.64.128 | 10.49.64.129 | .130 a .158 | 10.49.64.159 |
| Web_Apps (23) | 28 | /27 | 255.255.255.224 | 10.49.64.160 | 10.49.64.161 | .162 a .190 | 10.49.64.191 |
| NOC (33) | 10 | /28 | 255.255.255.240 | 10.49.64.192 | 10.49.64.193 | .194 a .206 | 10.49.64.207 |
| Enlace R-CENTRAL1 / R-CENTRAL2 | 2 | /30 | 255.255.255.252 | 10.49.64.0 | n/a | .1 y .2 | 10.49.64.3 |

Core_BD necesita 14 hosts más el gateway, es decir 15 direcciones, por eso es /27 y no /28.

**Dispositivos finales del Data Center (7 servidores):**

| Servidor | VLAN | Switch | IP | Máscara | Gateway |
|---|---|---|---|---|---|
| SRV-DB-1 | 13 | SW-DC-DIST | 10.49.64.130 | 255.255.255.224 | 10.49.64.129 |
| SRV-DB-2 | 13 | SW-DC-DIST | 10.49.64.131 | 255.255.255.224 | 10.49.64.129 |
| SRV-WEB-1 | 23 | SW-DC-DIST | 10.49.64.162 | 255.255.255.224 | 10.49.64.161 |
| SRV-WEB-2 | 23 | SW-DC-DIST | 10.49.64.163 | 255.255.255.224 | 10.49.64.161 |
| SRV-WEB-3 | 23 | SW-DC-ACC | 10.49.64.164 | 255.255.255.224 | 10.49.64.161 |
| SRV-NOC-1 | 33 | SW-DC-ACC | 10.49.64.194 | 255.255.255.240 | 10.49.64.193 |
| SRV-NOC-2 | 33 | SW-DC-ACC | 10.49.64.195 | 255.255.255.240 | 10.49.64.193 |

---

## 5. Área 1: Data Center y Sede Central

### 5.1 Topología física y mapeo lógico

```
R-CENTRAL1 --(10.49.64.0/30)-- R-CENTRAL2
 (Gi0/0/1)                      (Gi0/0/0)
                                (Gi0/0/1 troncal 802.1Q)
                                     |
                                SW-DC-ACC (2960, acceso, VTP client)
                                 Gi0/1 ==\\
                                 Gi0/2 ==// Po1 (LACP)
                                 Fa0/23-24 del 3560
                                SW-DC-DIST (3560, distribución, VTP server, root STP)
```

| Rol | Equipo | Observación |
|---|---|---|
| Router de redistribución | R-CENTRAL1 | Rutas estáticas hacia el Data Center, redistribuidas en OSPF |
| Router de acceso | R-CENTRAL2 | Router-on-a-Stick, solo rutas estáticas |
| Switch de distribución | SW-DC-DIST (3560) | VTP Server, root bridge de las VLANs 13, 23 y 33 |
| Switch de acceso | SW-DC-ACC (2960) | VTP Client, servidores NOC y WEB-3 |

**Conexiones físicas:**

| Equipo A | Puerto | Equipo B | Puerto | Tipo |
|---|---|---|---|---|
| R-CENTRAL1 | Gi0/0/1 | R-CENTRAL2 | Gi0/0/0 | Enlace L3 (10.49.64.0/30) |
| R-CENTRAL2 | Gi0/0/1 | SW-DC-ACC | Fa0/24 | **Troncal 802.1Q** |
| SW-DC-ACC | Gi0/1 | SW-DC-DIST | Fa0/23 | **EtherChannel Po1 (LACP)** |
| SW-DC-ACC | Gi0/2 | SW-DC-DIST | Fa0/24 | **EtherChannel Po1 (LACP)** |
| SW-DC-DIST | Fa0/1-2 | SRV-DB-1 y DB-2 | | Acceso VLAN 13 |
| SW-DC-DIST | Fa0/3-4 | SRV-WEB-1 y WEB-2 | | Acceso VLAN 23 |
| SW-DC-ACC | Fa0/1-2 | SRV-NOC-1 y NOC-2 | | Acceso VLAN 33 |
| SW-DC-ACC | Fa0/3 | SRV-WEB-3 | | Acceso VLAN 23 |

**Troncales 802.1Q:** Fa0/24 de SW-DC-ACC hacia R-CENTRAL2 y Po1 entre los dos switches.

### 5.2 Configuraciones principales

**SW-DC-DIST (3560): VTP Server, Rapid PVST+, root forzado y EtherChannel**

```
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
spanning-tree vlan 13,23,33 priority 4096
interface range FastEthernet0/23 - 24
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-protocol lacp
 channel-group 1 mode active
interface Port-channel1
 switchport trunk encapsulation dot1q
 switchport mode trunk
interface range FastEthernet0/1 - 2
 switchport mode access
 switchport access vlan 13
 spanning-tree portfast
interface range FastEthernet0/3 - 4
 switchport mode access
 switchport access vlan 23
 spanning-tree portfast
interface GigabitEthernet0/1
 shutdown
```

**SW-DC-ACC (2960): VTP Client y EtherChannel**

```
hostname SW-DC-ACC
spanning-tree mode rapid-pvst
vtp domain bantech49
vtp password cisco
vtp mode client
interface range GigabitEthernet0/1 - 2
 switchport mode trunk
 channel-protocol lacp
 channel-group 1 mode active
interface Port-channel1
 switchport mode trunk
interface FastEthernet0/24
 switchport mode trunk
interface range FastEthernet0/1 - 2
 switchport mode access
 switchport access vlan 33
 spanning-tree portfast
interface FastEthernet0/3
 switchport mode access
 switchport access vlan 23
 spanning-tree portfast
```

**R-CENTRAL2 (ISR 4331): Router-on-a-Stick y ruta por defecto estática**

```
hostname R-CENTRAL2
interface GigabitEthernet0/0/0
 ip address 10.49.64.2 255.255.255.252
 no shutdown
interface GigabitEthernet0/0/1
 no ip address
 no shutdown
interface GigabitEthernet0/0/1.13
 encapsulation dot1Q 13
 ip address 10.49.64.129 255.255.255.224
interface GigabitEthernet0/0/1.23
 encapsulation dot1Q 23
 ip address 10.49.64.161 255.255.255.224
interface GigabitEthernet0/0/1.33
 encapsulation dot1Q 33
 ip address 10.49.64.193 255.255.255.240
ip route 0.0.0.0 0.0.0.0 10.49.64.1
```

**R-CENTRAL1 (ISR 4331): rutas estáticas hacia el Data Center y redistribución en OSPF**

```
interface GigabitEthernet0/0/0
 shutdown
 no ip address
interface GigabitEthernet0/0/1
 ip address 10.49.64.1 255.255.255.252
 no shutdown
ip route 10.49.64.128 255.255.255.224 10.49.64.2
ip route 10.49.64.160 255.255.255.224 10.49.64.2
ip route 10.49.64.192 255.255.255.240 10.49.64.2
router ospf 1
 passive-interface GigabitEthernet0/0/1
 redistribute static subnets
```

### 5.3 Justificación de la arquitectura

**Por qué esta topología.** Los servidores se reparten en dos capas, un switch de distribución (3560) que concentra las bases de datos y los servidores web, y un switch de acceso (2960) para NOC y un servidor web. Un único router de acceso (R-CENTRAL2) hace el inter-VLAN por subinterfaces. Esto separa la función de conmutación de la de enrutamiento y permite que R-CENTRAL1 sea el único punto que conecta el Data Center con el backbone.

**Ventajas de tolerancia a fallos.** Los dos cables entre SW-DC-ACC y SW-DC-DIST forman un EtherChannel LACP. Si uno falla, el tráfico sigue por el otro sin que STP tenga que reconverger, porque el canal se ve como un solo enlace lógico. SW-DC-DIST es root bridge con prioridad 4096, así que la topología de STP es predecible.

**Cómo responde al contexto.** Los enlaces hacia las granjas de servidores manejan ráfagas masivas de tráfico y un solo cable no tiene el ancho de banda suficiente. El EtherChannel suma las capacidades de dos enlaces y reparte las conexiones entre ellos. El segmento queda aislado en rutas estáticas por la política de seguridad: R-CENTRAL2 no corre ningún protocolo dinámico y solo R-CENTRAL1 anuncia al núcleo las tres redes de servidores.

---

## 6. Área 2: Sede Occidente

### 6.1 Topología física y mapeo lógico

```
R-OCCIDENTE (Gi0/0) -- backbone (10.49.0.36/30)
R-OCCIDENTE (Gi0/1) -- troncal 802.1Q -- SW-OCC-1 (Gi0/1)
                                          Fa0/1-7: 7 PCs
```

| Rol | Equipo |
|---|---|
| Router de borde e inter-VLAN | R-OCCIDENTE (ISR 2911) |
| Switch principal y de acceso | SW-OCC-1 (2960) |

| Enlace | Tipo |
|---|---|
| SW-OCC-1 Gi0/1 con R-OCCIDENTE Gi0/1 | **Troncal 802.1Q** (VLANs 12, 22, 32 y 42) |
| SW-OCC-1 Fa0/1-2 | Acceso VLAN 12 (Cajas) |
| SW-OCC-1 Fa0/3-4 | Acceso VLAN 22 (Asesores) |
| SW-OCC-1 Fa0/5 | Acceso VLAN 32 (Gerencia) |
| SW-OCC-1 Fa0/6-7 | Acceso VLAN 42 (Seguridad) |

### 6.2 Configuraciones principales

**SW-OCC-1 (2960)**

```
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
interface GigabitEthernet0/1
 switchport mode trunk
interface range FastEthernet0/1 - 2
 switchport mode access
 switchport access vlan 12
 spanning-tree portfast
interface range FastEthernet0/3 - 4
 switchport mode access
 switchport access vlan 22
 spanning-tree portfast
interface FastEthernet0/5
 switchport mode access
 switchport access vlan 32
 spanning-tree portfast
interface range FastEthernet0/6 - 7
 switchport mode access
 switchport access vlan 42
 spanning-tree portfast
```

**R-OCCIDENTE (ISR 2911): Router-on-a-Stick y OSPF**

```
hostname R-OCCIDENTE
interface GigabitEthernet0/1
 no shutdown
interface GigabitEthernet0/1.12
 description Cajas
 encapsulation dot1Q 12
 ip address 10.49.16.1 255.255.255.192
interface GigabitEthernet0/1.22
 description Asesores
 encapsulation dot1Q 22
 ip address 10.49.16.65 255.255.255.192
interface GigabitEthernet0/1.32
 description Gerencia
 encapsulation dot1Q 32
 ip address 10.49.16.129 255.255.255.240
interface GigabitEthernet0/1.42
 description Seguridad
 encapsulation dot1Q 42
 ip address 10.49.16.145 255.255.255.240
router ospf 1
 network 10.49.16.0 0.0.0.255 area 0
 network 10.49.0.36 0.0.0.3 area 0
 passive-interface GigabitEthernet0/1.12
 passive-interface GigabitEthernet0/1.22
 passive-interface GigabitEthernet0/1.32
 passive-interface GigabitEthernet0/1.42
```

### 6.3 Justificación de la arquitectura

**Por qué esta topología.** Occidente es una agencia comercial con cuatro áreas funcionales. Un switch de acceso con un troncal hacia el router (Router-on-a-Stick) es la solución más simple de administrar: todo el enrutamiento entre VLANs está en un solo punto y agregar una VLAN solo requiere una subinterfaz nueva.

**Ventajas de tolerancia a fallos.** La segmentación en cuatro VLANs confina los broadcasts de cada área. Una falla o una tormenta en la VLAN de Asesores no afecta a las cajas, porque cada una es un dominio de broadcast distinto. Las interfaces de usuario están en modo pasivo en OSPF, de modo que no se envían hellos hacia las PCs.

**Cómo responde al contexto.** La necesidad operativa es una red fácilmente administrable y altamente segmentada para que un problema en el área de asesores no afecte las transacciones de las cajas. Cajas está en su propia VLAN y subred (/26), y el tráfico entre áreas solo cruza por el router, donde se pueden aplicar ACLs si se requiere.

**VTP.** Occidente tiene un único switch de acceso, por lo que VTP no es obligatorio según el enunciado.

---

## 7. Pruebas y evidencia

**Las imagenes se encuentran en la carpeta IMG**
\Redes1_2S_2026_G2\img\configuracion.png
\Redes1_2S_2026_G2\img\configurandoippc.png
\Redes1_2S_2026_G2\img\ping entre compus omogeneas.png
\Redes1_2S_2026_G2\img\ping entre otras vlans.png
\Redes1_2S_2026_G2\img\ping entre vlans.png
\Redes1_2S_2026_G2\img\ping mismas vlan.png

### 7.1 Occidente

| Prueba | Resultado esperado | Estado | Captura |
|---|---|---|---|
| `show vlan brief` en SW-OCC-1 | VLANs 12, 22, 32 y 42 con nombres y puertos | Verificada | `![](img/occ-vlan.png)` |
| `show interfaces trunk` en SW-OCC-1 | Gi0/1 en trunk con 12, 22, 32 y 42 | Verificada | `![](img/occ-trunk.png)` |
| `show ip interface brief` en R-OCCIDENTE | Cuatro subinterfaces en up/up | Verificada | `![](img/occ-ipbrief.png)` |
| Ping de PC Cajas a gateways `.1`, `.65`, `.129` y `.145` | 4/4 respuestas | Verificada (`.1`, `.65`, `.129`) | `![](img/occ-ping-gw.png)` |
| Ping entre VLANs (Cajas a Asesores) | TTL 127, 4/4 | Verificada | `![](img/occ-ping-intervlan.png)` |
| Ping de Occidente al Data Center (10.49.64.130) | 4/4 | **Pendiente** (ver sección 9) | |

### 7.2 Data Center

| Prueba | Resultado esperado | Estado | Captura |
|---|---|---|---|
| `show etherchannel summary` en SW-DC-ACC | `Po1(SU)` con `Gi0/1(P)` y `Gi0/2(P)` | Verificada | `![](img/dc-etherchannel-acc.png)` |
| `show etherchannel summary` en SW-DC-DIST | `Po1(SU)` con `Fa0/23(P)` y `Fa0/24(P)` | **Pendiente de captura** | |
| `show vtp status` en SW-DC-ACC | Client, dominio `bantech49`, VLANs 13, 23 y 33 aprendidas | Verificada | `![](img/dc-vtp-acc.png)` |
| `show vtp status` en SW-DC-DIST | Server, dominio `bantech49` | **Pendiente de captura** | |
| `show spanning-tree vlan 13` en SW-DC-DIST | Este switch es root | **Pendiente de captura** | |
| `show interfaces trunk` en SW-DC-ACC | Po1 y Fa0/24 en trunk con 1, 13, 23 y 33 | Verificada | `![](img/dc-trunk-acc.png)` |
| `show ip route` en R-CENTRAL2 | 3 rutas `C` de VLANs, 1 `C` del enlace, `S*` por defecto | Verificada | `![](img/dc-route-central2.png)` |
| `show ip route static` en R-CENTRAL1 | 3 rutas `S` vía 10.49.64.2 | Verificada | `![](img/dc-route-central1.png)` |
| Ping R-CENTRAL1 a R-CENTRAL2 (10.49.64.2) | Respuestas | Verificada | `![](img/dc-ping-routers.png)` |
| Pings entre servidores de VLANs distintas | Respuestas | **Pendiente de captura** | |
| Ping SRV-NOC-1 a 10.49.16.2 (Occidente) | Respuestas | **Pendiente** (ver sección 9) | |

### 7.3 Pruebas de conmutación por error

| Prueba | Procedimiento | Resultado esperado | Estado |
|---|---|---|---|
| Caída de un enlace del EtherChannel | Ping continuo desde SRV-NOC-1 a SRV-DB-1 y `shutdown` en `Gi0/1` de SW-DC-ACC | El ping continúa por `Gi0/2`. `show etherchannel summary` muestra `Gi0/1(D)` y `Gi0/2(P)` | **Pendiente** |
| Restauración | `no shutdown` en `Gi0/1` | Po1 recupera ambos miembros en `(P)` | **Pendiente** |

---

## 8. Problemas encontrados y soluciones

| Problema | Causa | Solución |
|---|---|---|
| Po1 caído en SW-DC-ACC | El EtherChannel se configuró en puertos que no coincidían con el cableado real. Los cables estaban en Gi0/1-2 (no Fa0/23-24) y el router en Fa0/24 | Se verificó con `show cdp neighbors` y se movió el `channel-group` a Gi0/1-2 y el trunk a Fa0/24 |
| Ping R-CENTRAL1 a R-CENTRAL2 fallaba con enlace up/up | La IP 10.49.64.1/30 estaba en Gi0/0/0, pero el cable a R-CENTRAL2 llegaba a Gi0/0/1 (10.49.64.5/30) | Se movió 10.49.64.1/30 a Gi0/0/1 y se apagó Gi0/0/0 |
| Cable sobrante en rojo entre R-CENTRAL1 y SW-DC-DIST | Enlace no contemplado en el diseño, con ambos puertos apagados | Se deshabilitó y se retiró el cable de la topología |
| Subinterfaces de R-OCCIDENTE con VLANs 453/304/105/126 | El `.pkt` traía el direccionamiento anterior | Se reemplazaron por las subinterfaces 12, 22, 32 y 42 |
| Ping Cajas a Gerencia y Seguridad fallaba | SW-OCC-1 conservaba las VLANs viejas tras una fusión de ramas | Se recrearon las VLANs 12, 22, 32 y 42, se reasignaron los puertos y se borraron las anteriores |
| VLANs sin nombre (`VLAN0012`, etc.) | Los comandos `name` no se aplicaron | Se reaplicaron y se verificó con `show vlan brief` |
| Ruta estática `10.49.0.0/16` en R-CENTRAL1 | Ruta heredada del backbone original que creaba un agujero negro al redistribuir | Se eliminó. R-CENTRAL1 aprende el resto de la red por OSPF |
| Router-id OSPF duplicado (4.4.4.4) en R-OCCIDENTE y R-CENTRAL1 | Configuración mezclada entre versiones del `.pkt` | Fijar un router-id único en cada router (R-OCCIDENTE = 5.5.5.5) |
| Pérdida de configuración al reabrir el `.pkt` | No se ejecutó `write memory` antes de guardar | Ejecutar `write memory` en cada equipo antes de guardar y subir el `.pkt` |

---

## 9. Estado actual y pendientes

### Completado

- [x] Área 1: switches, EtherChannel LACP, VTP, Rapid PVST+, inter-VLAN en R-CENTRAL2, rutas estáticas y redistribución en R-CENTRAL1.
- [x] Área 2: VLANs, troncal, Router-on-a-Stick y OSPF.
- [x] Conectividad local verificada en ambas áreas.

### Pendiente

- [ ] **Conectividad de extremo a extremo (Occidente al Data Center).** R-OCCIDENTE tiene vecino OSPF en FULL con 10.49.0.37, pero no recibe rutas `O` ni `O E2` del resto de la red. Se debe coordinar con el responsable del backbone para que el equipo en 10.49.0.37 y el siguiente salto (10.49.0.41) conozcan `10.49.64.128/27`, `10.49.64.160/27` y `10.49.64.192/28`, y para que R-CENTRAL1 tenga una ruta hacia `10.49.16.0/24`.
- [ ] Verificar que el router-id de R-OCCIDENTE sea único (`show ip protocols`).
- [ ] Capturas pendientes de SW-DC-DIST (`show etherchannel summary`, `show vtp status`, `show spanning-tree vlan 13`).
- [ ] Pruebas de ping entre VLANs del Data Center y pruebas de falla del EtherChannel.
- [ ] Capturas de `show ip route` de los routers frontera con las redes redistribuidas.
- [ ] Ejecutar `write memory` en todos los equipos y subir el `.pkt` final.

---

## 10. Flujo de trabajo en Git

- Rama de integración: `develop`.
- Cada integrante trabaja su parte en una rama propia y hace commits frecuentes con mensajes descriptivos.
- El archivo `.pkt` es binario y Git no puede fusionarlo. Una sola persona hace la integración del `.pkt` final, y antes de subirlo se ejecuta `write memory` en cada equipo y se verifica que no se perdió la configuración de ninguna área.

```
git switch -c feature/occidente-datacenter
git add .
git commit -m "Configuración Data Center y Occidente"
git push -u origin feature/occidente-datacenter
```
