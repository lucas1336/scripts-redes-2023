## Configuracion de TF Redes

### Indice

- [Configuracion de TF Redes](#configuracion-de-tf-redes)
  - [Indice](#indice)
  - [Piura](#piura)
    - [Configuracion del router Piura](#configuracion-del-router-piura)
    - [Configuracion del switch multi-capa 1](#configuracion-del-switch-multi-capa-1)
    - [Configuracion del switch multi-capa 2](#configuracion-del-switch-multi-capa-2)
    - [Configuracion del switch](#configuracion-del-switch)
  - [Comandos Por Separado](#comandos-por-separado)
    - [Seguridad basica](#seguridad-basica)
    - [Vlan](#vlan)
    - [DHCP](#dhcp)
    - [Ospf](#ospf)
    - [Autenticacion PPP](#autenticacion-ppp)
    - [Politicas de seguridad](#politicas-de-seguridad)
    - [Seguridad SSH](#seguridad-ssh)
    - [Configuracion de domain lookup](#configuracion-de-domain-lookup)
    - [Comandos de verificacion](#comandos-de-verificacion)
  - [Tablas de mascaras de subred](#tablas-de-mascaras-de-subred)



### Piura

#### Configuracion del router Lima

```cmd
en
conf t
hostname RTPIU1

interface GigabitEthernet0/0
ip address 172.24.254.29 255.255.255.252
ip ospd 100 area 0
no shutdown

interface Serial0/0/1
description LINK PIURA-LIMA
bandwidth 8192
ip address 172.24.254.6 255.255.255.252
encapsulation ppp
ppp authentication pap
ppp pap sent-username RTPIU1 password 0 grupo4
no shutdown


router ospf 100
router-id 2.2.2.2
network 172.24.254.4 0.0.0.3 area 0
network 172.24.254.28 0.0.0.3 area 0

username AdminPiura secret grupo4
username RTLIM1 secret grupo4



ip access-list extended PIURA-FILTRO-FTP
remark "Filtrar el acceso de TODOS del servicio FTP, excepto de Marketing"
permit tcp 172.24.32.64 0.0.0.31 host 172.24.33.38 eq 20
permit tcp 172.24.32.64 0.0.0.31 host 172.24.33.38 eq ftp
deny tcp any any eq 20
deny tcp any any eq ftp
line vty 0 15
access-class PIURA-FILTRO-FTP in
exit

access-list 1 permit host 172.24.32.3
line vty 0 15
access-class 1 in
```

#### Configuracion del switch multi-capa 1

```cmd
en
conf t
hostname MLSPIU1

vlan 10
name ADMINISTRACION
vlan 20
name LOGISTICA
vlan 30
name MARKETING
vlan 40
name VENTAS
vlan 50
name FINANZAS
vlan 60
name WIFI-CLIENTES
vlan 70
name WIFI-EJECUTIVOS
vlan 80
name SERVIDORES
vlan 99
name NATIVA

int range f0/2-9
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan all
exit

int f0/10
no sw
ip add 172.24.254.38 255.255.255.252
ip ospf 100 area 0

int vlan 10
ip add 172.24.32.1 255.255.255.192
no shutdown
int vlan 20
ip add 172.24.32.129 255.255.255.224
no shutdown
int vlan 30
ip add 172.24.32.65 255.255.255.192
no shutdown
int vlan 40
ip add 172.24.32.193 255.255.255.224
no shutdown
int vlan 50
ip add 172.24.32.161 255.255.255.224
no shutdown
int vlan 60 
ip add 172.24.32.241 255.255.255.240
no shutdown
int vlan 70
ip add 172.24.33.1 255.255.255.240
no shutdown
int vlan 80
ip add 172.24.33.33 255.255.255.248
no shutdown
int vlan 99
ip add 172.24.99.65 255.255.255.224
no shutdown
exit

ip default-gateway 172.24.99.65
ip routing

int vlan 10
ip ospf 100 area 0
int vlan 20
ip ospf 100 area 0
int vlan 30
ip ospf 100 area 0
int vlan 40
ip ospf 100 area 0
int vlan 50
ip ospf 100 area 0
int vlan 60
ip ospf 100 area 0
int vlan 70
ip ospf 100 area 0
int vlan 80
ip ospf 100 area 0


ip dhcp pool pool-vlan10
network 172.24.32.0 255.255.255.192
default-router 172.24.32.1
dns-server 172.24.33.35
domain-name NETWORKPIONEERS.PIURA.COM
exit
ip dhcp pool pool-vlan20
network 172.24.32.128 255.255.255.224
default-router 172.24.32.129
dns-server 172.24.33.35
domain-name NETWORKPIONEERS.PIURA.COM
exit
ip dhcp pool pool-vlan30
network 172.24.32.64 255.255.255.192
default-router 172.24.32.65
dns-server 172.24.33.35
domain-name NETWORKPIONEERS.PIURA.COM
exit
ip dhcp pool pool-vlan40
network 172.24.32.192 255.255.255.224
default-router 172.24.32.193
dns-server 172.24.33.35
domain-name NETWORKPIONEERS.PIRA.COM
exit
ip dhcp pool pool-vlan50
network 172.24.32.160 255.255.255.224
default-router 172.24.32.161
dns-server 172.24.33.35
domain-name NETWORKPIONEERS.PIURA.COM
exit
ip dhcp pool pool-vlan60
network 172.24.32.240 255.255.255.240
default-router 172.24.32.241
dns-server 172.24.33.35
domain-name NETWORKPIONEERS.PIURA.COM
exit
ip dhcp pool pool-vlan70
network 172.24.33.0 255.255.255.240
default-router 172.24.33.1
dns-server 172.24.33.35
domain-name NETWORKPIONEERS.PIURA.COM
exit

router ospf 100
network 172.24.32.0 0.0.0.63 area 0
network 172.24.32.128 0.0.0.31 area 0
network 172.24.32.64 0.0.0.63 area 0
network 172.24.32.192 0.0.0.31 area 0
network 172.24.32.160 0.0.0.31 area 0
network 172.24.32.240 0.0.0.15 area 0
network 172.24.33.0 0.0.0.15 area 0
network 172.24.33.32 0.0.0.7 area 0
network 172.24.99.64 0.0.0.31 area 0
network 172.24.254.28 0.0.0.3 area 0
passive-interface f0/2
passive-interface f0/3
passive-interface f0/4
passive-interface f0/5
passive-interface f0/6
passive-interface f0/7
passive-interface f0/8
passive-interface f0/9
exit

ip access-list extended PIURA-FILTRO-WEB
remark "Filtrar el acceso de la VLAN de Ventas del servicio Web (80, 443), el resto debe pasar"
deny tcp 172.24.32.192 0.0.0.63 172.24.33.36 0.0.0.0 eq 80
deny tcp 172.24.32.192 0.0.0.63 172.24.33.36 0.0.0.0 eq 443
permit ip any any
exit
int vlan 40
ip access-group PIURA-FILTRO-WEB in
exit

ip access-list extended PIURA-FILTRO-CORREO
remark "Filtrar el acceso de la VLAN de Finanzas del servicio CORREO (25, 110 y 143), el resto debe pasar"
deny tcp 172.24.32.160 0.0.0.31 172.24.33.34 0.0.0.0 eq 25
deny tcp 172.24.32.160 0.0.0.31 172.24.33.34 0.0.0.0 eq 110
deny tcp 172.24.32.160 0.0.0.31 172.24.33.34 0.0.0.0 eq 143
permit ip any any
exit
int vlan 50
ip access-group PIURA-FILTRO-CORREO in
exit

ip access-list extended PIURA-FILTRO-FTP
remark "Filtrar parcialmente el acceso de la VLAN de Marketing del servicio FTP."
deny tcp 172.24.32.64 0.0.0.31 host 172.24.33.38 eq 20
deny tcp 172.24.32.64 0.0.0.31 host 172.24.33.38 eq 21
permit tcp any any eq 20
permit tcp any any eq 21
exit
line vty 0 15
access-class PIURA-FILTRO-FIP
exit
int vlan 30
ip access-group PIURA-FILTRO-FTP in
exit

access-list 1 permit host 172.24.32.3
ip access-list extended PIURA-FILTRO-SSH
remark "Filtrar el servicio SSH a todos, excepto el ADMINISTRADOR"
permit tcp host 172.24.32.3 any eq 22
permit tcp host 172.24.32.3 any eq telnet
deny tcp any any eq 22
deny tcp any any eq telnet
permit ip any any

#### Configuracion del switch

```cmd
en
conf t
hostname SWPIU8

vlan 10
name ADMINISTRACION
vlan 20
name LOGISTICA
vlan 30
name MARKETING
vlan 40
name VENTAS
vlan 50
name FINANZAS
vlan 60
name WIFI-CLIENTES
vlan 70
name WIFI-EJECUTIVOS
vlan 80
name SERVIDORES
vlan 99
name NATIVA

int vlan 99
ip add 172.24.99.67 255.255.255.224
no shutdown
exit
ip default-gateway 172.24.99.65
```

### Comandos Por Separado/Genéricos

#### Seguridad basica

```cmd
line console 0
password grupo4
login
line vty 0 15
password grupo4
login
exit
enable secret grupo4
banner motd % *** Solo personal del equipo 4 con la debida autorización, accedera al dispositivo *** %
```

#### Vlan

Configuracion en el switch multi-capa 1

```cmd
vlan 10
name ADMINISTRACION
vlan 20
name LOGISTICA
vlan 99
name NATIVA
exit

int range f0/2-9
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan all
exit

int vlan 10
ip add 172.24.16.1 255.255.255.128
no shutdown
int vlan 20
ip add 172.24.16.129 255.255.255.192
no shutdown
int vlan 99
ip add 172.24.99.33 255.255.255.224
no shutdown
exit

ip default-gateway 172.24.99.33
ip routing
```

Configuracion en el switch

```cmd
vlan 10
name ADMINISTRACION
vlan 20
name LOGISTICA
vlan 99
name NATIVA

int vlan 99
ip add 172.24.99.50 255.255.255.224
no shutdown
exit
ip default-gateway 172.24.99.33
```

#### DHCP

Configuracion en el switch multi-capa 1

```cmd
ip dhcp pool pool-vlan10
network 172.24.16.0 255.255.255.128
default-router 172.24.16.1
dns-server 172.24.17.230
domain-name NETWORKPIONEERS.LIMA.COM
exit
ip dhcp pool pool-vlan20
network 172.24.16.128 255.255.255.192
default-router 172.24.16.129
dns-server 172.24.17.230
domain-name NETWORKPIONEERS.LIMA.COM
exit
```

#### Ospf

Configuracion en el router

```cmd
router ospf 100
router-id 1.1.1.1
network 100.50.50.0 0.0.0.3 area 0
network 172.24.254.4 0.0.0.3 area 0
pasive-interface s0/0/0
default-information originate
exit
```

Configuracion en el switch multi-capa

```cmd
int vlan 10
ip ospf 100 area 0
int vlan 20
ip ospf 100 area 0

router ospf 100
network 172.24.16.0 0.0.0.127 area 0
network 172.24.16.128 0.0.0.63 area 0

passive-interface f0/8
passive-interface f0/9
exit
```

#### Autenticacion PPP

Configuracion en el router

```cmd
username RTPIU1 secret grupo4

int s0/0/1
encapsulation ppp
ppp authentication pap
ppp pap sent-username RTLIM1 password grupo4
```

> La primera linea es el usuario con el que se va a autenticar el router remoto. En el router remoto, se realiza la misma configuracion, pero intercambiando los nombres de usuario.

#### Politicas de seguridad

Denegar acceso al servicio telnet a todos los usuarios, excepto al administrador

```cmd
ip access-list extended LIMA-FILTRO-TELNET
remark "Filtrar el acceso de TODOS del servicio TELNET, excepto el Administrador"
permit tcp host 172.24.16.4 any eq telnet
deny tcp any any eq telnet
exit
line vty 0 15
access-class LIMA-FILTRO-TELNET in
exit
```

Denegar el acceso al servicio FTP a la VLAN de Marketing

```cmd
ip access-list extended LIMA-FILTRO-FTP
remark "Filtrar el acceso de la VLAN de Marketing del servicio FTP (20, 21), el resto debe pasar"
deny tcp 172.24.16.192 0.0.0.63 172.24.17.231 0.0.0.0 eq 20
deny tcp 172.24.16.192 0.0.0.63 172.24.17.231 0.0.0.0 eq 21
permit ip any any
exit
int vlan 30
ip access-group LIMA-FILTRO-FTP in
exit
```

Denegar el acceso al servicio WEB a la VLAN de Logistica

```cmd
ip access-list extended LIMA-FILTRO-WEB
remark "Filtrar el acceso de la VLAN de Losgistica del servicio WEB (80, 443), el resto debe pasar"
deny tcp 172.24.16.128 0.0.0.63 172.24.17.230 0.0.0.0 eq 80
deny tcp 172.24.16.128 0.0.0.63 172.24.17.230 0.0.0.0 eq 443
permit ip any any
exit
int vlan 20
ip access-group LIMA-FILTRO-WEB in
exit
```

#### Seguridad SSH

```cmd
username AdminLima secret grupo4
enable secret grupo4
ip domain-name NETWORKPIONEERS.LIMA.COM
ip ssh authentication-retries 3
ip ssh time-out 90
crypto key generate rsa
1024
ip ssh version 2
line vty 0 14
transport input all 
login local
exit
line console 0
login local
do copy run start
exit
access-list 1 permit host 172.24.16.4
line vty 0 15
access-class 1 in
exit
```

Para acceder se ingresa al CMD y se escribe el comando `ssh -l AdminLima 172.24.254.21`

#### Configuracion de domain lookup

```cmd
no ip domain-lookup
```

#### Comandos de verificacion

```cmd
show ip interface brief
show ip route
show interfaces vlan
do show run
do show history
do copy run start
```

### Tablas de mascaras de subred

| CIDR Notation |    Subnet Mask    | Wildcard Mask |
| :-----------: | :---------------: | :-----------: |
|      /24      |  `255.255.255.0`  |  `0.0.0.255`  |
|      /25      | `255.255.255.128` |  `0.0.0.127`  |
|      /26      | `255.255.255.192` |  `0.0.0.63`   |
|      /27      | `255.255.255.224` |  `0.0.0.31`   |
|      /28      | `255.255.255.240` |  `0.0.0.15`   |
|      /29      | `255.255.255.248` |   `0.0.0.7`   |
|      /30      | `255.255.255.252` |   `0.0.0.3`   |

---

[Regresar al inicio](#indice)

//002

