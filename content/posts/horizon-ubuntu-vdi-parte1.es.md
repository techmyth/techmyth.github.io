---
title: "VMware: Cómo crear imágenes de Linux para VDI en Horizon 8 2303"
author: 'Jonathan Colon Feliciano'
date: 2023-04-11T08:39:31-04:00
draft: false
tags:
  - "VMware"
  - "Horizon"
  - "HashiCorp"
  - "Packer"
  - "Linux"
  - "VDI"
---

En esta ocasión estaré hablando sobre como utilizar `HashiCorp` `packer` para crear una imagen VM máster que podemos utilizar como platilla. Una ventaja de este proceso es que esta imagen puede ser utilizada para publicar un Pool en VMware Horizon. Aunque existen varios `HowTo` de como crear automáticamente una plantilla con `packer` todos los artículos que he visto están orientados a Windows 10/11 y no así para Linux. Es por esta razón, que me di a la tarea de crear este artículo.

Es importante mencionar que VMware ofrece varios ejemplos de como crear una plantilla para los distintos sistemas operativos que se pueden automatizar con `packer`. Les dejo aquí el enlace:

<https://github.com/hashicorp/packer-plugin-vsphere>

Para este artículo utilizaré Ubuntu Linux 22.04 como ejemplo, pero también es posible automatizar cualquier versión de Linux soportada por VMware Horizon para escritorios en Linux.

Sistemas operativos Linux compatibles con Horizon Agent:

| Linux Distribution    |      Architecture      |
|----------|:-------------:|
| Ubuntu 20.04 and 22.04 |  x64 |
| Debian 10.13 and 11.5 |    x64   |
| Red Hat Enterprise Linux Workstation 7.9+ | x64|
| Red Hat Enterprise Linux Server 7.9+ | x64|
| Debian 10.13 and 11.5 |    x64   |
| SUSE Linux Enterprise Desktop (SLED) 15 SP3 and 15 SP4 |    x64   |
| SUSE Linux Enterprise Server (SLES) 15 SP3 and 15 SP4 |    x64   |

Para comenzar, he creado una plantilla de `packer` para Ubuntu 22.04 que voy a utilizar para este ejemplo. Pueden acceder este código desde Github en el siguiente enlace:

<https://github.com/rebelinux/packer-ubuntu-vsphere-horizon-iso>

El primer paso es clonar el repositorio de Github localmente. En mi caso estoy utilizando Linux en mi computadora principal pero también puede utilizarse Windows 10/11 para este ejemplo.

Para clonar el repositorio utilizaremos el comando `git clone "https://github.com/rebelinux/packer-ubuntu-vsphere-horizon-iso`

#### Paso 1: Clonar repositorio packer-ubuntu-vsphere-horizon-iso

```bash
[rebelinux@PC ~]$ git clone "https://github.com/rebelinux/packer-ubuntu-vsphere-horizon-iso"
Cloning into 'packer-ubuntu-vsphere-horizon-iso'...
remote: Enumerating objects: 93, done.
remote: Counting objects: 100% (93/93), done.
remote: Compressing objects: 100% (67/67), done.
remote: Total 93 (delta 51), reused 63 (delta 24), pack-reused 0
Receiving objects: 100% (93/93), 287.93 KiB | 1009.00 KiB/s, done.
Resolving deltas: 100% (51/51), done.
```

#### Ejemplo: Contenido de la plantilla de packer

```bash
[rebelinux@PC ~]$ ls "packer-ubuntu-vsphere-horizon-iso"
build-2204.ps1  files  README.md  ubuntu.pkr.hcl
build-2204.sh   http   setup      variables.auto.pkrvars.hcl.sample
[rebelinux@PC ~]$
```

Luego que tengamos el repositorio en nuestro directorio local es necesario modificar las variables pertinentes de nuestro ambiente de VMware vSphere.

#### Paso 2: Renombrar el archivo de variables

```bash
[rebelinux@PC ~]$ cd "packer-ubuntu-vsphere-horizon-iso/"
[rebelinux@PC ~]$ mv "variables.auto.pkrvars.hcl.sample" "variables.auto.pkrvars.hcl"
[rebelinux@PC ~]$ 
```

#### Paso 3: Modificar las variables

Esta sección del archivo define las variables del ambiente de vSphere:

```bash
# Name or IP of you vCenter Server
vsphere_server          = "vcenter.lab.local"

# vsphere username
vsphere_username        = "administrator@vsphere.local"

# vsphere password
vsphere_password        =```````

# vsphere datacenter name
vsphere_datacenter      = "LAB-VSI-DC"

# vsphere cluster name
vsphere_cluster           = "RegionHQ-MGMT"

# vsphere folder name
vsphere_folder           = "Horizon Lab"

# vsphere network
vsphere_network         = "LAB-VM-Network"

# vsphere datastore
vsphere_datastore       = "SSD-VM-HIGH-CAPACITY-PERF-KN"

# vsphere VM Name
vsphere_vm_name         = "hz-tpl-ubuntu"
```

Esta porción del archivo define los parámetros básicos de la VM como lo son las credenciales y los script que modifican la imagen:

```bash
# final clean up script
shell_scripts           = [
    "./setup/packages.sh",
    "./setup/desktop_postinstall.sh",
    "./setup/ad_domain_join.sh",
    "./setup/horizon_agent_install.sh",
    "./setup/cleanup.sh"
    ]

# SSH username
build_username            = "godadmin"

# SSH password
build_password            = "godadmin"

# To generate an encypted password for user-data use the following command:
# mkpasswd -m sha-512
build_password_encrypted = "$6$rounds=4096$Y0SjrsU5WHubYJvb$0BJhswGEAokE2OqlRFTgiUhJnquzDt2hAnrb3.g3DNTATZ01VLNbxlLRLMLk.PTHiMeP8fUg9WfVx.HeL7e8E0"

# ISO Objects
iso_path = ["[HDD-VM-ISO-LOW-PERF] /ISO/Linux/ubuntu-22.04-live-server-amd64.iso"]
```

Esta porción del archivo define los parámetros del ambiente de dominio para añadir la VM a Active Directory:

##### Nota: Para que funcione SSO es requisito añadir la VM a un dominio de Active Directory

```bash
# NTP Server
ntp_server = "xx.xx.xx.xx"
timezone = "America/Puerto_Rico"

# AD Domain
ad_domain = "PHARMAX.LOCAL"

# AD Domain join password
join_password =`````

# AD Domain join username
join_username = "Administrator"
```

Para finalizar esta parte del archivo define el nombre del paquete de instalación del agente de VMware Horizon:

```bash
# Horizon Agent install files
horizon_agent_file = "VMware-horizonagent-linux-x86_64-2303-8.9.0-21434177.tar.gz"


# Uncommnet this if using vcenter as web server
// horizon_agent_datastore = "HDD-VM-ISO-LOW-PERF"
// horizon_agent_path = "/ISO/VMWARE/Horizon/"
```

#### Paso 4: Copiar el archivo del agente de Horizon

En este paso es necesario copiar el archivo de instalación del agente de Horizon a la carpeta `files`.

```bash
├── build-2204.ps1
├── build-2204.sh
├── "files"
│   ├── Place_Here_Horizon_Agent_Files.txt
│   └── "VMware-horizonagent-linux-x86_64-2303-8.9.0-21434177.tar.gz"
├── http
│   ├── meta-data
│   └── user-data.pkrtpl.hcl
├── packerlog.txt
├── README.md
├── setup
│   ├── ad_domain_join.sh
│   ├── cleanup.sh
│   ├── desktop_postinstall.sh
│   ├── horizon_agent_install.sh
│   └── packages.sh
├── ubuntu.pkr.hcl
├── variables.auto.pkrvars.hcl
└── variables.auto.pkrvars.hcl.sample
```

#### Paso 5: Inicializar packer

Luego es necesario inicializar `packer init .` desde la consola de comandos y validar la configuración con el comando `packer validate .`

```bash
[rebelinux@rebelpc]$ packer init .
Installed plugin github.com/hashicorp/vsphere v1.1.2 in ".config/packer/plugins/github.com/hashicorp/vsphere/packer-plugin-vsphere_v1.1.2_x5.0_linux_amd64"
[rebelinux@rebelpc]$ packer validate .
"The configuration is valid."
[rebelinux@rebelpc]$ 
```

#### Paso 6: Crear una plantilla de máquina virtual

Por último, para comenzar a generar la plantilla para la VM de Ubuntu es necesario ejecutar el comando `packer build .` En el caso de esta plantilla de `packer` se creo el archivo `build-2204.sh` para hacer el proceso mas simple de ejecutar.

```bash
[rebelinux@rebelpc]$ ./build-2204.sh 
vsphere-iso.ubuntu: output will be in this color.

==> vsphere-iso.ubuntu: Creating CD disk...
    vsphere-iso.ubuntu: Warning: creating filesystem with (nonstandard) Joliet extensions
    vsphere-iso.ubuntu: but without (standard) Rock Ridge extensions.
    vsphere-iso.ubuntu: It is highly recommended to add Rock Ridge
    vsphere-iso.ubuntu: Setting input-charset to 'UTF-8' from locale.
    vsphere-iso.ubuntu: Total translation table size: 0
    vsphere-iso.ubuntu: Total rockridge attributes bytes: 0
    vsphere-iso.ubuntu: Total directory bytes: 0
    vsphere-iso.ubuntu: Path table size(bytes): 10
    vsphere-iso.ubuntu: Max brk space used 0
    vsphere-iso.ubuntu: 181 extents written (0 MB)
    vsphere-iso.ubuntu: Done copying paths from CD_dirs
==> vsphere-iso.ubuntu: Uploading packer3491041722.iso to packer_cache/packer3491041722.iso
==> vsphere-iso.ubuntu: Creating VM...
==> vsphere-iso.ubuntu: Customizing hardware...
==> vsphere-iso.ubuntu: Mounting ISO images...
==> vsphere-iso.ubuntu: Adding configuration parameters...
==> vsphere-iso.ubuntu: Set boot order...
==> vsphere-iso.ubuntu: Power on VM...
==> vsphere-iso.ubuntu: Waiting 3s for boot...
==> vsphere-iso.ubuntu: Typing boot command...
==> vsphere-iso.ubuntu: Waiting for IP...
==> vsphere-iso.ubuntu: IP address: 192.168.7.136
==> vsphere-iso.ubuntu: Using SSH communicator to connect: 192.168.7.136
==> vsphere-iso.ubuntu: Waiting for SSH to become available...
==> vsphere-iso.ubuntu: Connected to SSH!
==> vsphere-iso.ubuntu: Uploading ./files/ => /tmp/
==> vsphere-iso.ubuntu: Provisioning with shell script: ./setup/packages.sh
    vsphere-iso.ubuntu: ===> Updating Apt
    vsphere-iso.ubuntu: > Disable invalid v4l2loopback driver...
    vsphere-iso.ubuntu: ===> Installing additional packages
    vsphere-iso.ubuntu: ===> Updating MLocate database
==> vsphere-iso.ubuntu: Provisioning with shell script: ./setup/desktop_postinstall.sh
    vsphere-iso.ubuntu: ===> Enable the boot splash
    vsphere-iso.ubuntu: ===> Enable ssh password auth and permit root login
    vsphere-iso.ubuntu: ===> Add user to sudoers file
    vsphere-iso.ubuntu: ===> Change /etc/sudoers.d/PHARMAX.LOCAL permissions
==> vsphere-iso.ubuntu: Provisioning with shell script: ./setup/ad_domain_join.sh
    vsphere-iso.ubuntu: ===> Pointing NTP Server to PHARMAX.LOCAL Domain
    vsphere-iso.ubuntu: ===> Join PHARMAX.LOCAL Active Directory Domain
    vsphere-iso.ubuntu: Password for <sensitive>:
    vsphere-iso.ubuntu: ===> Disable use_fully_qualified_names in AD Login
    vsphere-iso.ubuntu: ===> Disable use_fully_qualified_names in AD Login
    vsphere-iso.ubuntu: ===> Enable Automatic home directory creation for network users
==> vsphere-iso.ubuntu: Provisioning with shell script: ./setup/horizon_agent_install.sh
    vsphere-iso.ubuntu: ===> Extracting VMware Horizon Agent files to /tmp/hzagentdir directory
    vsphere-iso.ubuntu: ===> Creating /tmp/hzagentdir directory
    vsphere-iso.ubuntu: ===> Downloading v4l2loopback driver files
    vsphere-iso.ubuntu: ===> Downloading VHCI-HCD driver files
    vsphere-iso.ubuntu: ===> Extracting V4L2Loopback driver files
    vsphere-iso.ubuntu: ===> Patching V4L2Loopback driver files
    vsphere-iso.ubuntu: ===> Extracting V4L2Loopback driver files
    vsphere-iso.ubuntu: Building v4l2-loopback driver...
    vsphere-iso.ubuntu: make -C /lib/modules/`uname -r`/build M=/tmp/v4l2loopback-0.12.5 KCPPFLAGS="" modules
    vsphere-iso.ubuntu: make[1]: Entering directory '/usr/src/linux-headers-5.15.0-70-generic'
==> vsphere-iso.ubuntu: Skipping BTF generation for /tmp/v4l2loopback-0.12.5/v4l2loopback.ko due to unavailability of vmlinux
==> vsphere-iso.ubuntu: arch/x86/Makefile:142: CONFIG_X86_X32 enabled but no binutils support
    vsphere-iso.ubuntu:   INSTALL /lib/modules/5.15.0-70-generic/extra/v4l2loopback.ko
    vsphere-iso.ubuntu:   SIGN    /lib/modules/5.15.0-70-generic/extra/v4l2loopback.ko
    vsphere-iso.ubuntu:   DEPMOD  /lib/modules/5.15.0-70-generic
==> vsphere-iso.ubuntu: Warning: modules_install: missing 'System.map' file. Skipping depmod.
    vsphere-iso.ubuntu: make[1]: Leaving directory '/usr/src/linux-headers-5.15.0-70-generic'
    vsphere-iso.ubuntu:
    vsphere-iso.ubuntu: SUCCESS (if you got 'SSL errors' above, you can safely ignore them)
    vsphere-iso.ubuntu:
    vsphere-iso.ubuntu: ===> Installing v4l2loopback-ctl
    vsphere-iso.ubuntu: install -p -m 755 -d "/usr/local/bin"
    vsphere-iso.ubuntu: install -p -m 755 utils/v4l2loopback-ctl "/usr/local/bin"
    vsphere-iso.ubuntu: ===> Extracting VHCI-HCD driver files
    vsphere-iso.ubuntu: ===> Patching VHCI-HCD driver files
    vsphere-iso.ubuntu: patching file Makefile
    vsphere-iso.ubuntu: patching file usb-vhci-dump-urb.c
    vsphere-iso.ubuntu: patching file usb-vhci.h
    vsphere-iso.ubuntu: patching file usb-vhci-hcd.c
    vsphere-iso.ubuntu: patching file usb-vhci-hcd.h
    vsphere-iso.ubuntu: patching file usb-vhci-iocifc.c
    vsphere-iso.ubuntu: ===> Compiling VHCI-HCD driver files
    vsphere-iso.ubuntu: make testconfig
    vsphere-iso.ubuntu: ===> Installing Horizon Agent
    vsphere-iso.ubuntu: ===> Looking for vhci-hcd driver status
    vsphere-iso.ubuntu: ===> Found vhci-hcd driver, enabling Horizon USB redirection
    vsphere-iso.ubuntu: ===> Looking for V4L2Loopback driver status
    vsphere-iso.ubuntu: ===> Found V4L2Loopback driver, enabling Horizon Real-Time Audio-Video
    vsphere-iso.ubuntu: ===> Using install options: -A yes -U yes -a yes --webcam
    vsphere-iso.ubuntu: /tmp/hzagentdir
    vsphere-iso.ubuntu: The installation will install VMware Horizon Agent on your computer.
    vsphere-iso.ubuntu: Installation Start ...
    vsphere-iso.ubuntu:
    vsphere-iso.ubuntu: If you have any questions or issues regarding Linux VDI, please start a discussion at https://communities.vmware.com, and we will respond to you as soon as possible.
    vsphere-iso.ubuntu: You must restart your system for the configuration changes made to the VMware Horizon Agent to take effect.
    vsphere-iso.ubuntu:
    vsphere-iso.ubuntu: Installation done
    vsphere-iso.ubuntu:
    vsphere-iso.ubuntu: ===> Setting SSSD authentication (OfflineJoinDomain=sssd)
==> vsphere-iso.ubuntu: Provisioning with shell script: ./setup/cleanup.sh
    vsphere-iso.ubuntu: > Cleaning all audit logs ...
    vsphere-iso.ubuntu: > Cleaning SSH keys ...
    vsphere-iso.ubuntu: ===> Remove default filesystem and related tools not used with the suggested
    vsphere-iso.ubuntu: ===> Remove other packages present by default in Ubuntu Server but not normally present in Ubuntu Desktop
    vsphere-iso.ubuntu: > Cleaning apt-get ...
    vsphere-iso.ubuntu: > Disable Ubuntu AutoUpdate...
    vsphere-iso.ubuntu: > Cleaning the machine-id ...
    vsphere-iso.ubuntu: > Cleaning cloud-init
    vsphere-iso.ubuntu: datasource_list: [ VMware, NoCloud, ConfigDrive ]
==> vsphere-iso.ubuntu: Executing shutdown command...
==> vsphere-iso.ubuntu: Deleting Floppy drives...
==> vsphere-iso.ubuntu: Eject CD-ROM drives...
==> vsphere-iso.ubuntu: Deleting CD-ROM drives...
==> vsphere-iso.ubuntu: Creating snapshot...
Build 'vsphere-iso.ubuntu' finished after 22 minutes 14 seconds.

==> Wait completed after 22 minutes 14 seconds

==> Builds finished. The artifacts of successful builds are:
--> vsphere-iso.ubuntu: hz-tpl-ubuntu
[rebelinux@rebelpc]$
```

#### Ejemplo: VM creada en vCenter

![Text](/img/2023/horizon-ubuntu-vdi-2404/packer_vm_info.webp#center)

#### Ejemplo: Proceso de Instalación de Ubuntu automatizado

![Text](/img/2023/horizon-ubuntu-vdi-2404/packer_ubuntu.webp#center)

#### Ejemplo: Imagen de la VM ya creada

![Text](/img/2023/horizon-ubuntu-vdi-2404/packer_vm_status.webp#center)

Ahora pasaremos a probar la imagen de la VM de Ubuntu Linux 22.04 utilizando un Pool de VMware Horizon ya creado para este propósito.

![Text](/img/2023/horizon-ubuntu-vdi-2404/horizon-packer-ubuntu-image.webp#center)

#### Ejemplo: Pool con la VM ya creada

![Text](/img/2023/horizon-ubuntu-vdi-2404/horizon-vm-ic-status.webp#center)

#### Video de VMware Horizon Client

{{< youtube fzCuAddi-BM >}}

En la segunda parte de esta serie de artículos estaré explicando como modificar la plantilla de `packer` y como editar los scripts que configuran la imagen.

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/F1F8DEV80)

![Text](/img/hasta-luego-5937ba.webp#center)
