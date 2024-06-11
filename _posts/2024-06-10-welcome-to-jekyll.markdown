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
## Introducción

En este proyecto se va a realizar la infraestructura de una empresa, en este caso será una empresa encargada de suministros escolares, material de oficina y reprografía (Gestión y reparación de máquinas fotocopiadoras).

Para ello nos guiarse en los planos de la nave donde se encuentra la empresa y buscaremos **la mejor opción posible para realizar nuestro proyecto**.

![alt image](Capturas/Imagen-1.png)

Como se ve en el plano de generado en GNS3 nos encontramos con una nave de **dos plantas**. En la primera planta nos encontramos con toda la parte del **almacen** donde se aloján los suministros escolares. El **departamento de técnicos de las máquinas fotocopiadoras**, donde se encargarán de la reparación y configuración de las máquinas fotocopiadoras y la zona donde ubicaremos la **DMZ**.

En la primera planta se ubicará **el departamento de admininstración y finanzas**.

