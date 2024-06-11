---
layout: post
title:  "Montaje de la infraestructura de red"
date:   2024-06-05 02:33:18 +0200
---

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

#### Creación de las VLANs

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
|Switch Técnicos|![alt image](https://github.com/sepp30000/sepp30000.github.io/blob/main/_posts/Capturas/Switch%20T%C3%A9cnicos.png?raw=true)|
|Switch Administración|![alt image](https://github.com/sepp30000/sepp30000.github.io/blob/main/_posts/Capturas/Switch%20administraci%C3%B3n.png?raw=true)|
|Switch Almacen|![alt image](https://github.com/sepp30000/sepp30000.github.io/blob/main/_posts/Capturas/Switch%20Almacen.png?raw=true)|
|Switch Enlace|![alt image](https://github.com/sepp30000/sepp30000.github.io/blob/main/_posts/Capturas/Siwtch%20enlace.png?raw=true)|



Ahora en el router se crean las VLANs

```bash
/interface/vlan
add name=tecnicos vlan-id=10 interface=ether3 
add name=almacen vlan-id=20 interface=ether3 
add name=administracion vlan-id=30 interface=ether3
```

![alt image](https://github.com/sepp30000/sepp30000.github.io/blob/main/_posts/Capturas/vlan1.png?raw=true)

Y las interfaces de red

```bash
/ip/address
add address=192.168.21.1/24 interface=tecnicos  
add address=192.168.22.1/24 interface=almacen    
add address=192.168.23.1/24 interface=administracion 
```

![alt image](https://github.com/sepp30000/sepp30000.github.io/blob/main/_posts/Capturas/vlan2.png?raw=true)

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

![alt image](https://github.com/sepp30000/sepp30000.github.io/blob/main/_posts/Capturas/VLAN3.png?raw=true)

Finalmente se configura para que todas las interfaces internas salgan por la que está conectada a internet(**ether1**).

```bash
/ip/firewall/nat
add action=masquerade chain=srcnat out-interface=ether1 
```

#### Conexión entre routers

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

#### Port-Forwarding

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

### VPN

Ahora que están los routers configurados y totalmente funcionales, llega el momento a montar una VPn. Una VPN es una herramienta de red que nos permite hacer una extensión de nuestra red local. Esto es muy útil porque gracias a esto se podrá entrar a nuestra red interna desde cualquier lugar. Además, solo estará abierto el puerto de la VPN desde afuera ya que solo se puede entrar a los servidores desde la red interna como se ha realizado anteriormente en la configuración de los routers, proporcionandonos, una mayor seguridad al proyecto.

La VPN elegida para esta ocasión es **Wireguard**. **Wireguard** es una VPN creada en 2015, de código abierto y bastante popular en la comunidad. Una de las principales razones por las que hemos elegido **Wireguard** es la integración de esta VPN dentro de Mikrotik de manera nativa dentro de su S.O. *RouterOS* dando la fácilidad de configuración dentro del router. Dicho esto comienza la configuración.

#### Actualizar el Router y creación de la interfaz de Wireguard

El primer paso será realizar una actualización del router. Para eso comienza en:

```bash
# Busca si hay actualizaciones
/system package update check-for-updates
# Actualiza el S.O.
/system package update install
```

Después de tener el sistema operativo actualizado, tocará crear la interfaz de la VPN y su red interna.

***¡Muy importante!***. **Está configuración ha sido realizada con fines explicativos. Las claves públicas y privadas mostradas en este proyecto, ya no existen porque representarían un agujero de seguridad importante.**

```bash
# Esto a parte de hacer la interfaz de la vpn,creará una private y public key del servidor.
/interface/wireguard add name=wg0 listen-port=51820
```

![alt image](https://github.com/sepp30000/sepp30000.github.io/blob/main/_posts/Capturas/Wireguard-crearinterfaz.png?raw=true)

```bash
# Creando la red de la VPN
/ip/adress add address=192.168.23.2/24 network=192.168.23.0 interface=wg0
```

![alt image](https://github.com/sepp30000/sepp30000.github.io/blob/main/_posts/Capturas/Wireguard-Direccion.png?raw=true)

Con esto ya estaría creada la interfaz de **Wireguard**

#### Configuración del Firewall

Continua con la configuración del firewall del router con el objetivo de que no solo se pueda entrar a nuestra red interna vía VPN.

```bash
# Bloquea cualquier acceso a la red
/ip/firewall/filter add chain=forward action=drop
# Permite el acceso tanto UDP como TCP el puerto configurado de nuestra VPN
/ip/firewall/filter add chain=input action=accept protocol=udp dst-port=51820
/ip/firewall/filter add chain=input action=accept protocol=tcp dst-port=51820
# Habilita el acceso a internet al igual que hemos echo con las vlan creadas antes
/ip/firewall/nat add chain=srcnat action=masquerade out-interface=wg0
```

#### Creción de la peer

Ahora es el momento de la creación de la parte de *peer*. La primera parte será descargar [el cliente de wireguard](https://www.wireguard.com/install/) (En este caso el cliente de MACOSX). Después de instalar la aplicación, hay que hacer click en **crear un tunel vacio**, después se dentro de esta opción el nombre de la interfaz será el de la VPN "wg0" y se copia la clave pública.

![alt image](https://github.com/sepp30000/sepp30000.github.io/blob/main/_posts/Capturas/Wireguard-Cliente.png?raw=true)

![alt image](https://github.com/sepp30000/sepp30000.github.io/blob/main/_posts/Capturas/Wireguard-Tunel-vacio.png?raw=true)

Ahora en la web de *Mikrotik* en el apartado **Wireguard/Peers** se crea una nueva peer.

![alt image](https://github.com/sepp30000/sepp30000.github.io/blob/main/_posts/Capturas/Wireguard-Peer.png?raw=true)

- Interface: Interfaz que utliza la peer. En este caso "wg0".

- Public Key: La clave pública del cliente de Wireguard.

- Allowed Address: La red que utilizará el cliente.

Después de configurar la Peer se generá un código QR que se puede utilizar en el cliente para móviles. Pero en este caso la configuración se realizará de forma manual.

![alt image](https://github.com/sepp30000/sepp30000.github.io/blob/main/_posts/Capturas/Wireguard%20QR.png?raw=true)

#### Configuración del cliente

Finalmente falta la configuración del cliente. 

```bash
# Repesenta la configuración del equipo cliente
[Interface]
PrivateKey = [Generada por el cliente]
# Direccion que ocupará el equipo
Address = 192.168.23.3/24
#DNS Será la puerta de enlace del router
DNS = 192.168.23.2
# Respecto al servidor
[Peer]
PublicKey = [Clave Pública del servidor]
# Así permite cualquier ip de donde esté conectado el equipo
AllowedIPs = 0.0.0.0/0
# Donde tiene que llegar el equipo, si hubiera un dns dinamico sería esa direccion más el puerto
Endpoint = 192.168.1.38:51820
# Manda paquetes para saber si sigue conectado
PersistentKeepalive = 10
```

![alt image](https://github.com/sepp30000/sepp30000.github.io/blob/main/_posts/Capturas/Wireguard-ClienteConfigurado.png?raw=true)


Y finalmente se prueba la configuración

![alt image](https://github.com/sepp30000/sepp30000.github.io/blob/main/_posts/Capturas/Wireguard-ClienteFuncionando.png?raw=true)

![alt image](https://github.com/sepp30000/sepp30000.github.io/blob/main/_posts/Capturas/Wireguard-Funcionando.png?raw=true)

---
