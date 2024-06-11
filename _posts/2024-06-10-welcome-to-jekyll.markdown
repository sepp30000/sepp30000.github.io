---
layout: post
title:  "Proyecto Final"
date:   2024-06-09 02:33:18 +0200
categories: jekyll update
---

# Proyecto Final de CFGS de José Ramón Peris Murcia 2º ASIR
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>

![Imagen](Capturas/Redes-de-FIbra-optica.jpg)

<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>

<center>José Ramón Peris</center>
<center>Fecha: 11-06-2024</center>

---

## Indice
1. [Introducción](#Introducción)
2. [¿Qué es lo que busca el cliente?](#que-es-lo-que-nos-pide-el-cliente)
4. [Presupuesto del proyecto](#presupuesto-del-proyecto)
5. [Montaje de la infraestructura de red](#montaje-de-la-infraestructura-de-red)    
    1. [Configuración de la red](#configuración-de-la-red)
    2. [Configuración de los routers](#configuración-de-los-routers)
    3. [VPN](#VPN)
6. [Sistemas de almacenamiento](#sistemas-de-almacenamiento)
    1. [Instalación de Docker](#instalación-de-docker)
    2. [WebDAV](#webdav)
    3. [FTP](#ftp)
1. [CRUD en Postgre](#crud-en-postgres)
    1. [Creación de la BBDD](#creación-de-la-bbdd)
    1. [Programación en PHP](#programación-en-php)
<br>

## Introducción

En este proyecto se va a realizar la infraestructura de una empresa, en este caso será una empresa encargada de suministros escolares, material de oficina y reprografía (Gestión y reparación de máquinas fotocopiadoras).

Para ello nos guiarse en los planos de la nave donde se encuentra la empresa y buscaremos **la mejor opción posible para realizar nuestro proyecto**.

![alt image](Capturas/Imagen-1.png)

Como se ve en el plano de generado en GNS3 nos encontramos con una nave de **dos plantas**. En la primera planta nos encontramos con toda la parte del **almacen** donde se aloján los suministros escolares. El **departamento de técnicos de las máquinas fotocopiadoras**, donde se encargarán de la reparación y configuración de las máquinas fotocopiadoras y la zona donde ubicaremos la **DMZ**.

En la primera planta se ubicará **el departamento de admininstración y finanzas**.

## ¿Que es lo que nos pide el cliente?

*El cliente busca que realicemos una actualización de su infraestructura de red y una digitalización del almacen, de la zona de técnicos y administración.*

Para ello se le planteará las siguientes propuestas:

- Levantar una infraestructura de red mediante la configuración de routers Mikrotik que realizará la salida de internet, firewall y portforwarding.

- La implantación de una aplicación de tareas para que los técnicos puedan realizar un seguimiento de las máquinas desde que entran al taller hasta que salen del mismo.

- Desarrollo de una solución de almacenamiento de datos que permita la no dependencia de servicios de nube como Dropbox o Google Drive.

- La construcción de una DMZ que permita tener dos servidores encargados de los servicios implementados

- La implantación de una VPN que permita acceder a estos servicios.

La empresa acepta la propuesta y da por comenzar el proyecto

## Presupuesto del proyecto

Después del estudio que hemos realizado en la nave, se encuentran con que el *tema de infraestructura de cableado a sido montada con anterioridad al tratarse de una nave moderna.* Por lo tanto se presupuesta tanto los equipos nuevos, la DMZ, y el tratamiento de switches, routers, etc.

Aquí nos encontramos con el presupuesto que le hemos realizado a la empresa con todos sus enlaces en la web de **PcComponentes**:

| Modelo | Cantidad | Precio Unidad | Enlace |
|:------:|:--------:|:-------------:|------|
| Armario Rack 19" 22U 600x600 (Para DMZ) |  1 |   413,67€   |[Comprar](https://www.pccomponentes.com/armario-rack-19-22u-600x600?utm_source=531573&utm_medium=afi&utm_campaign=shopforward.nl&sv1=affiliate&sv_campaign_id=531573&awc=20982_1716742105_4386269844ea01d5a9218daa611daf51&utm_term=deeplink&utm_content=25082.Cj0KCQjwu8uyBhC6ARIsAKwBGpSQPnuLyLwCtWH3D-_6RvKMMFQPTFny4pVcGFAZUzhE5W9CqmC1W5AaAsp_EALw_wcB)|
| VidaXL Armario Rack 19" 12U 600x640mm (Para administración) | 1   | 148,98€  |[Comprar](https://www.pccomponentes.com/vidaxl-armario-rack-19-12u-600x640mm?utm_source=531573&utm_medium=afi&utm_campaign=shopforward.nl&sv1=affiliate&sv_campaign_id=531573&awc=20982_1716742054_91b2a72c9893094b43aeeccc211d676d&utm_term=deeplink&utm_content=25082.Cj0KCQjwu8uyBhC6ARIsAKwBGpRdj6UrKEbh_rgSWqzv1DDvmSA3wRlq9MszeB_RlaJcYq-nWxEXADwaAqzTEALw_wcB)|
| VidaXL Armario Rack 19" 6U 600x450x375mm  | 3   | 96,99€ |[Comprar](https://www.pccomponentes.com/vidaxl-armario-rack-19-6u-600x450x375mm?utm_source=531573&utm_medium=afi&utm_campaign=shopforward.nl&sv1=affiliate&sv_campaign_id=531573&awc=20982_1716742063_d5438b74e1ac4e3cba6a08ad1bab3d2c&utm_term=deeplink&utm_content=27932.Cj0KCQjwu8uyBhC6ARIsAKwBGpR11uxnDf4huC_xA1SgIRFAKPqfaK_BeLPBwxaF_hS2rL4J5qdqCeMaAgmyEALw_wcB)|
|Mikrotik RB1100AHx4 Router Ethernet 13 Puertos RJ45 Gigabit PoE | 2 | 314,39€ |[Comprar](https://www.pccomponentes.com/mikrotik-rb1100ahx4-router-ethernet-13-puertos-rj45-gigabit-poe)|
| TP-Link TL-SG1024DE Switch 24 Puertos Gigabit | 5 | 107,43€ |[Comprar](https://www.pccomponentes.com/tp-link-tl-sg1024de-switch-24-puertos-gigabit?utm_source=531573&utm_medium=afi&utm_campaign=shopforward.nl&sv1=affiliate&sv_campaign_id=531573&awc=20982_1716742507_4e25f87295e75b24d972fe659c5435b5&utm_term=deeplink&utm_content=11894.Cj0KCQjwu8uyBhC6ARIsAKwBGpQNMki_9xVS9lmNUM4YVDgASzJW0sPfgHJ1tzq1_vXMTUv00pNjXpsaAmmBEALw_wcB)|
| Equip 326424 Patch panel 24 Puertos Cat 6 | 5 | 79,65€ |[Comprar](https://www.pccomponentes.com/equip-326424-panel-de-parcheo-24-puertos-cat-6)|
| Dell PowerEdge R350 Intel Xeon E-2314/16GB/600GB  | 3 | 1699,00€ | [Comprar](https://www.pccomponentes.com/dell-poweredge-r350-intel-xeon-e-2314-16gb-600gb?utm_source=531573&utm_medium=afi&utm_campaign=shopforward.nl&sv1=affiliate&sv_campaign_id=531573&awc=20982_1716742606_5a39c8a66215f2a7a5db017c7dcee381&utm_term=deeplink&utm_content=717.Cj0KCQjwu8uyBhC6ARIsAKwBGpQMeBHuC80RNCs4UzuRrZbxDQ-PsbGrZ2maCkwp8ppcUHiTgIGlu1oaAhuMEALw_wcB)|
| HP Pavilion All-in-One 27-ca2008ns Intel Core i5-13400T/16GB/512GB SSD/27" (Equipos de trabajo) | 7 | 899,01€ |[Comprar](https://www.pccomponentes.com/hp-pavilion-all-in-one-27-ca2008ns-intel-core-i5-13400t-16gb-512gb-ssd-27?s_kwcid=AL!14405!3!!!!x!!&gad_source=1&gclid=Cj0KCQjwu8uyBhC6ARIsAKwBGpQvVMRSji4wCRDVypftl-USqlGeUqZLbilSEix2VwgitTBOOjG1aFwaAnOwEALw_wcB)|
|Dell Vostro 3520 Intel Core i5-1235U/16GB/512GB SSD/15.6" (Portatiles de backup)|3|639,00€ |[Comprar](https://www.pccomponentes.com/dell-vostro-3520-intel-core-i5-1235u-16gb-512gb-ssd-156?utm_source=624709&utm_medium=afi&utm_campaign=www.twenga-solutions.com&sv1=affiliate&sv_campaign_id=624709&awc=20981_1716742876_68a3fdd1d2d05c9f5a7d5f14ed57f07d&utm_term=deeplink&utm_content=ac5f26529b84508e0911daf7189909c5)|
|-|-|-|-|
|-|-|**Total: 15872,87€ IVA Incuido**|-|

## Montaje de la infraestructura de red

El primer paso a realizar es el montaje de la infraestructura de la red. Este consistirá en los siguientes puntos:

- El montaje de un sistema de routers **Mikrotik** que se encargue del *enrutamiento de la infraestructura de la empresa.*

- Implantación de un servicio de **DNSMasq** responsable de *la asignación de los dns y el dhcp.*

- Configuración del **portforwarding** de los routers, buscando *una mayor seguridad y la redirección de los servicios a nuestra DMZ.*

- Compra y preparación de equipos para almacen, técnicos así como administración y dmz.


### Configuración de la red

Ahora toca la configuración de la red. En este caso se realizará la configuración con unos routers de la marca **Mikrotik** que utilizan el sistema operativo *Router OS* que permite una gran configuración y personalización.

Para ello se realiza la configuración de los **2 routers** que se han comprado.

- Uno será el router de entrada/salida de internet.

- El otro será un router será el encargado de separar a la  **DMZ** del resto de equipos. Haciendo una separación clara entre ellas.

A parte de todo esto, se creará diferentes VLANs que aportará una mayor seguridad, eficiencia y mejor gestión de las redes.

### Configuración de los routers

Para configurar los routers se dividirá en dos partes:

- Configuración de la **red interna de la empresa**, en la que entrará todo el apartado de las *VLAN* y la salida a internet de los equipos.

- Configuración de la **DMZ**, donde se alojará los servidores y comprende el apartado de comuncicación entre los dos routers y la redirección de puertos.

1. Creación de las VLANs

Cada departamento de la empresa va a tener su propia red:

| Nombre | Dirección | Mascara | 
|:------:|:--------:|:-------------:|
|Administración y finanzas|192.168.23.0|24|
|Técnicos|192.168.22.0|24|
|Almacen|192.168.21.0|24|

Para ello se hará uso de las VLANs. Esto permite crear redes lógicas independientes dentro de la misma red física.

Todo esto se realizará en el Router-Entrada. La conexión entre routers será para más adelante. Para crear las VLANs lo primero será la configuración de los switches.

| Nombre | Configuración |
|:------:|:--------:|
|Switch Técnicos|![alt image](Capturas/Switch%20Técnicos.png)|
|Switch Administración|![alt image](Capturas/Switch%20administración.png)|
|Switch Almacen|![alt image](Capturas/Switch%20Almacen.png)|
|Switch Enlace|![alt image](Capturas/Siwtch%20enlace.png)|



Ahora en el router se crean las VLANs

```bash
/interface/vlan
add name=tecnicos vlan-id=10 interface=ether3 
add name=almacen vlan-id=20 interface=ether3 
add name=administracion vlan-id=30 interface=ether3
```

![alt image](Capturas/vlan1.png)

Y las interfaces de red

```bash
/ip/address
add address=192.168.21.1/24 interface=tecnicos  
add address=192.168.22.1/24 interface=almacen    
add address=192.168.23.1/24 interface=administracion 
```

![alt image](Capturas/vlan2.png)

Después de esto, los equipos pueden tener ip fija o que un servidor dhcp les asigne una dirección ip. Mikrotik dispone de uno y su configuración es:(Solo se enseña la de uno, el resto es igual)

```bash
/ip/dhcp-server
setup

Select interface to run DHCP server on 

dhcp server interface:  
administracion     almacen     ether1     ether2     ether3     ether4     ether5     ether6     ether7     ether8     tecnicos   
dhcp server interface: tecnicos 
Select network for DHCP addresses 

dhcp address space: 192.168.21.0/24  
Select gateway for given network 

gateway for dhcp network: 192.168.21.1 
Select pool of ip addresses given out by DHCP server 

addresses to give out: 192.168.21.2-192.168.21.254, 
Select DNS servers 

dns servers: 8.8.8.8,1.1.1.1            
Select lease time 

lease time: 1800 
```

![alt image](Capturas/VLAN3.png)

Finalmente se configura para que todas las interfaces internas salgan por la que está conectada a internet(**ether1**).

```bash
/ip/firewall/nat
add action=masquerade chain=srcnat out-interface=ether1 
```

2. Conexión entre routers

Ahora que está creada la red interna, el siguiente objetivo la conexión entre el **router-salida** y **router-DMZ**.

En router-entrada se crea interfaz que conecte con DMZ

```bash
/ip/address
add address=192.168.10.1/24 interface=ether2
```

En router-DMZ se crea la interfaz que conecte con entrada

```bash
/ip/address
add address=192.168.10.2/24 interface=ether1
```

La salida a internet de manera similar a entrada,además de crear la red interna de la DMZ

```bash
/ip/firewall/nat/
add action=masquerade chain=srcnat out-interface=ether1
/ip/address
add address=192.168.11.1/24 interface=ether2
```

Y la puerta de enlace entre los routers más los dns

```bash
/ip/route
add gateway=192.168.10.1
/ip/dns
set servers=8.8.8.8,1.1.1.1
```

3. Port-Forwarding

Ahora que los routers se comunican, llega el momento de la redirección de puertos. El objetivo es que desde fuera de la red interna no se pueda acceder a la DMZ. Por lo tanto, el **Port-forwarding** es una gran opción.

```bash
# 192.168.21.1
/ip/firewall/nat
# WebDAV
add chain=dstnat dst-address=192.168.21.1 protocol=tcp dst-port=80 action=dst-nat to-addresses=192.168.10.2 to-ports=80
add chain=dstnat dst-address=192.168.21.1 protocol=tcp dst-port=443 action=dst-nat to-addresses=192.168.10.2 to-ports=443
# Postgre
add chain=dstnat dst-address=192.168.21.1 protocol=tcp dst-port=5432 action=dst-nat to-addresses=192.168.10.2 to-ports=5432
# FTP
add chain=dstnat dst-address=192.168.21.1 protocol=tcp dst-port=21 action=dst-nat to-addresses=192.168.10.2 to-ports=21
# Puertos dinamicos
add chain=dstnat dst-address=192.168.21.1 protocol=tcp dst-port=30000-31000 action=dst-nat to-addresses=192.168.10.2 to-ports=30000-31000
# Teamvierwer
add chain=dstnat dst-address=192.168.21.1 protocol=tcp dst-port=5938 action=dst-nat to-addresses=192.168.10.2 to-ports=5938
# ssh
add chain=dstnat dst-address=192.168.21.1 protocol=tcp dst-port=22 action=dst-nat to-addresses=192.168.10.2 to-ports=22
# Pgadmin
add chain=dstnat dst-address=192.168.21.1 protocol=tcp dst-port=5938 action=dst-nat to-addresses=192.168.10.2 to-ports=5050
# PHP-Apache
add chain=dstnat dst-address=192.168.21.1 protocol=tcp dst-port=5938 action=dst-nat to-addresses=192.168.10.2 to-ports=8080

# 192.168.22.1
/ip/firewall/nat
# WebDAV
add chain=dstnat dst-address=192.168.22.1 protocol=tcp dst-port=80 action=dst-nat to-addresses=192.168.10.2 to-ports=80
add chain=dstnat dst-address=192.168.22.1 protocol=tcp dst-port=443 action=dst-nat to-addresses=192.168.10.2 to-ports=443
# Postgre
add chain=dstnat dst-address=192.168.22.1 protocol=tcp dst-port=5432 action=dst-nat to-addresses=192.168.10.2 to-ports=5432
# FTP
add chain=dstnat dst-address=192.168.22.1 protocol=tcp dst-port=21 action=dst-nat to-addresses=192.168.10.2 to-ports=21
# Puertos dinamicos
add chain=dstnat dst-address=192.168.22.1 protocol=tcp dst-port=30000-31000 action=dst-nat to-addresses=192.168.10.2 to-ports=30000-31000
# Teamvierwer
add chain=dstnat dst-address=192.168.22.1 protocol=tcp dst-port=5938 action=dst-nat to-addresses=192.168.10.2 to-ports=5938
# ssh
add chain=dstnat dst-address=192.168.22.1 protocol=tcp dst-port=22 action=dst-nat to-addresses=192.168.10.2 to-ports=22
# Pgadmin
add chain=dstnat dst-address=192.168.22.1 protocol=tcp dst-port=5938 action=dst-nat to-addresses=192.168.10.2 to-ports=5050
# PHP-Apache
add chain=dstnat dst-address=192.168.22.1 protocol=tcp dst-port=5938 action=dst-nat to-addresses=192.168.10.2 to-ports=8080

# 192.168.23.1
/ip/firewall/nat
# WebDAV
add chain=dstnat dst-address=192.168.23.1 protocol=tcp dst-port=80 action=dst-nat to-addresses=192.168.10.2 to-ports=80
add chain=dstnat dst-address=192.168.23.1 protocol=tcp dst-port=443 action=dst-nat to-addresses=192.168.10.2 to-ports=443
# Postgre
add chain=dstnat dst-address=192.168.23.1 protocol=tcp dst-port=5432 action=dst-nat to-addresses=192.168.10.2 to-ports=5432
# FTP
add chain=dstnat dst-address=192.168.23.1 protocol=tcp dst-port=21 action=dst-nat to-addresses=192.168.10.2 to-ports=21
# Puertos dinamicos
add chain=dstnat dst-address=192.168.23.1 protocol=tcp dst-port=30000-31000 action=dst-nat to-addresses=192.168.10.2 to-ports=30000-31000
# Teamvierwer
add chain=dstnat dst-address=192.168.23.1 protocol=tcp dst-port=5938 action=dst-nat to-addresses=192.168.10.2 to-ports=5938
# ssh
add chain=dstnat dst-address=192.168.23.1 protocol=tcp dst-port=22 action=dst-nat to-addresses=192.168.10.2 to-ports=22
# Pgadmin
add chain=dstnat dst-address=192.168.23.1 protocol=tcp dst-port=5938 action=dst-nat to-addresses=192.168.10.2 to-ports=5050
# PHP-Apache
add chain=dstnat dst-address=192.168.23.1 protocol=tcp dst-port=5938 action=dst-nat to-addresses=192.168.10.2 to-ports=8080
```


