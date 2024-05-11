---
title: "HomeLab: QuickBoot en vSphere 7 update 2 con ESXi Virtual"
date: 2021-04-07T08:39:31-04:00
author: 'Jonathan Colon Feliciano'
draft: false
tags:
  - "VMware"
---

En este blog, estaré validando si la función de "ESXi QuickBoot" trabaja en modo Nested osea correr ESXi en una VM. Primero que nada debemos saber que es QuickBoot y que requerimientos tiene. Para esto utilizaré como referencia la documentación del portal de VMWare:

> Quick Boot es una función de vSphere que acelera el proceso de actualización de un servidor ESXi. Un reinicio normal implica un ciclo de alimentación completo que requiere la inicialización del firmware y de los dispositivos. Quick Boot optimiza la ruta de reinicio para evitar esto, ahorrando un tiempo considerable del proceso de actualización.

#### [Understanding ESXi Quick Boot Compatibility](https://kb.vmware.com/s/article/52477)

![Text](/img/QuickBoot_WorkFlow.webp#center)

Requerimientos:

- La plataforma del manufacturero debe ser compatible
- Todos los controladores de los dispositivos deben ser compatibles

Limitaciones en vSphere 7.0

- El TPM esté desactivado

Limitaciones en vSphere 6.7:

- El TPM esté desactivado
- No hay VM con dispositivos en "passthrough" configurados
- No hay controladores vmklinux cargados en el ESXi

La documentación de VMware incluye un listado de manufactureros de servidores compatibles con ésta tecnología para referencia puede ver en enlace del [knowledge Base](https://kb.vmware.com/s/article/52477).

Para ESXi 7.0 o versiones más recientes, puede comprobar la compatibilidad del `hardware` aquí:

- Quick Boot en Dell EMC
- Quick Boot en HPE
- Quick Boot en Cisco
- Quick Boot en Fujitsu
- Quick Boot en Lenovo

En éste laboratorio haré pruebas con ESXi virtual utilizando "Nested Virtualization". Es importante aclarar que este es un ambiente de prueba. Para utilizar esta tecnología  en ambientes de producción es necesario activar la opción de QuickBoot desde `VMware Update Manager`  o el renombrado `Lifecycle Manager`. Si les interesa les dejo un vídeo aquí

[Updates Installation with vSphere ESXi QuickBoot](https://youtu.be/FTwglwgDWAE)

```text
[root@comp-01a:~] esxcli hardware platform get
Platform Information
   UUID: 0x27 0xa8 0x30 0x42 0xa3 0xf9 0x54 0xcf 0xe2 0xa3 0x10 0x1d 0xfd 0xb8 0x21 0xd0
   Product Name: VMware7,1
   Vendor Name: VMware, Inc.
   Serial Number: VMware-42 30 a8 27 f9 a3 cf 54-e2 a3 10 1d fd b8 21 d0
   Enclosure Serial Number: None
   BIOS Asset Tag: No Asset Tag
   IPMI Supported: false
[root@comp-01a:~]
```

El primer paso para activar QuickBoot es validar la compatibilidad del servidor. Para esto es necesario conectarse por SSH al servidor de ESXi para ejecutar el comando `loadESXCheckCompat.py`. En mi caso el comando válido que mi plataforma es compatible.

```text
[root@comp-01a:~] /usr/lib/vmware/loadesx/bin/loadESXCheckCompat.py
This system is compatible with Quick Boot.
[root@comp-01a:~]
```

Una vez validamos que el servidor es compatible podemos activar la función de QuickBoot con el comando `loadESXEnable -e`.

```text
[root@comp-01a:~] /bin/loadESXEnable -e
INFO: LoadESX Enabled
INFO: Precheck options:
INFO:   All prechecks are enabled.
[root@comp-01a:~]
```

El paso final para activar la función de QuickBoot sería cargar la configuración de `QuickLaunch` utilizando el comando `loadESX.py`.

```text
[root@comp-01a:~] /usr/lib/vmware/loadesx/bin/loadESX.py
DEBUG: LoadESX scripts are up to date.
INFO: Enabling QuickLaunch kernel preload
DEBUG: Using a ramdisk at /tmp/loadESX for intermediate storage
DEBUG: SYSTEM STORAGE IS ENABLED
INFO: Target version: 7.0.2-0.0.17867351
DEBUG: Install boot module "vmware_e.v00" with size 0x2ec40
DEBUG: Install boot module "vmware_f.v00" with size 0x1e3582f
DEBUG: Install boot module "vsan.v00" with size 0x25f08a8
DEBUG: Install boot module "vsanheal.v00" with size 0x80ebe0
DEBUG: Install boot module "vsanmgmt.v00" with size 0x18957cc
DEBUG: Install boot module "xorg.v00" with size 0x358c40
DEBUG: Install boot module "gc.v00" with size 0xe077
DEBUG: Install boot module "imgdb.tgz" with size 0x192800
DEBUG: Install boot module "state.tgz" with size 0x21a00
INFO: loadESX is ready ...
INFO: Performing QuickLaunch kernel preload...
[root@comp-01a:~] 
```

Les incluyo unos vídeos de prueba para que puedan visualizar cuan rápida es la tecnología de QuickBoot al reiniciar el servidor.

VMware vSphere Normal Boot:
{{< youtube o0rvzKIu4DY >}}

VMware vSphere QuickBoot:
{{< youtube RGEA2pUSaus >}}

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/F1F8DEV80)
