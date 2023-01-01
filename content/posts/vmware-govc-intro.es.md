---
title: 'Govc: CLI de vSphere basado en govmomi, una introducción'
date: '2022-08-12T10:50:00-04:00'
tags:
    - vSphere
    - Golang
---

Hola a todos,

Esta vez estaré hablando de cómo utilizar **Govc**, una alternativa de CLI desarrollada en el lenguaje de programación **[GO](https://go.dev/)**, esta CLI está diseñada para ser una alternativa fácil de usar al Web GUI y muy adecuada para tareas de automatización. Esta herramienta es basada en **govmomi**, que es una librería para GO utilizada para interactuar con el Application Programming Interface (API) de VMware vSphere

#### Nota: Las versiones de vSphere que han llegado al final de su soporte pueden funcionar, pero no cuentan con soporte oficial.

Aquí tienes el enlace para que puedas ver las características de esta herramienta.

- <https://github.com/vmware/govmomi/blob/main/govc/README.md>

Esta herramienta se puede instalar de muchas maneras pero para este tutorial utilizaré la versión compilada. Para utilizar otro método de instalación puede validar la documentación [Govc](https://github.com/vmware/govmomi/tree/main/govc#installation).

El primer paso sería descargar la herramienta desde el portal [Github](https://github.com/vmware/govmomi/releases). En mi caso utilizaré la versión x86_64 que es la que corresponde a mi sistema Linux pero ésta herramienta provee soporte para la mayoría de los Sistemas Operativos más utilizados.

![Text](/img/2022/vmware-govc-intro/govc_download_x86.webp#center)

Luego de descargar el paquete es necesario descomprimir el contenido para poder tener acceso al archivo compilado.

```sh
[bla@blabla Downloads]$ ls g*
govc_Linux_x86_64.tar.gz
[bla@blabla Downloads]$ cp govc_Linux_x86_64.tar.gz govc
[bla@blabla Downloads]$ cd govc/
[bla@blabla govc]$ ls
govc_Linux_x86_64.tar.gz
[bla@blabla govc]$ tar -zxf govc_Linux_x86_64.tar.gz -v
CHANGELOG.md
LICENSE.txt
README.md
govc
[bla@blabla govc]$ ls
CHANGELOG.md  **govc**  govc_Linux_x86_64.tar.gz  LICENSE.txt  README.md
[bla@blabla govc]$ 
```

En la carpeta que hemos descomprimido podemos ver el archivo **govc** que es el binario principal de la herramienta. Para hacer pruebas con **govc** me conectaré al vCenter de mi HomeLab que utiliza la dirección FQDN **vcenter-01v.pharmax.local**.

```sh
[bla@blabla Downloads]$ ./govc about -u administrator@vsphere.local:SECUREPASSWORD@vcenter-01v.pharmax.local
FullName:     VMware vCenter Server 8.0.0 build-20920323
Name:         VMware vCenter Server
Vendor:       VMware, Inc.
Version:      8.0.0
Build:        20920323
OS type:      linux-x64
API type:     VirtualCenter
API version:  8.0.0.1
Product ID:   vpx
UUID:         43f86c8f-1a6d-44c8-b6ac-6ec33a36d141
[bla@blabla govc]$ 
```

#### Nota: Esta herramienta asume que el certificado HTTPS puede ser validado con la cadena de confianza. Si el certificado es "Self Sign" es necesario configurar la variable GOVC_INSECURE=1

En este artículo les mostré una introducción sobre la herramienta **Govc** en los proximos articulos estaré entrando en detalle de como utilizar esta herramienta para agilizar los procesos de automatización en nuestra infraestructura de vSphere.

#### Hasta Luego Amigos!
