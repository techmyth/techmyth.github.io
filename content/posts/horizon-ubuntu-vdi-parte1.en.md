---
title: "VMware: How to create Linux VDI Images for for Horizon 8 2303"
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

<!---[Parte 1]({{< ref "/posts/horizon-ubuntu-vdi-parte1.es.md" >}} "Cómo utilizar Ubuntu Linux 22.04 como VDI en Horizon 8 2303")-->

This time I am going to talk about how to use **HashiCorp** `packer` to create a VM master image that can be used as a template to publish a Pool in VMware Horizon. Although there are many HowTo's on how to automatically create a template with `packer`, all the ones I have seen are oriented to Windows 10/11 and not for Linux. It is for this reason, that I gave myself the task of creating this post.

It is important to mention that VMware offers several examples of how to create a template for the different operating systems that can be automated with `packer`. Here is the link:

<https://github.com/hashicorp/packer-plugin-vsphere>

This article uses Ubuntu Linux 22.04 as an example but it is also possible to automate any Linux version supported by VMware Horizon for Linux desktops.

| Linux Distribution    |      Architecture      |
|----------|:-------------:|
| Ubuntu 20.04 and 22.04 |  x64 |
| Debian 10.13 and 11.5 |    x64   |
| Red Hat Enterprise Linux Workstation 7.9+ | x64|
| Red Hat Enterprise Linux Server 7.9+ | x64|
| Debian 10.13 and 11.5 |    x64   |
| SUSE Linux Enterprise Desktop (SLED) 15 SP3 and 15 SP4 |    x64   |
| SUSE Linux Enterprise Server (SLES) 15 SP3 and 15 SP4 |    x64   |

To start, I have created a `packer` template for Ubuntu 22.04 that I am going to use for this demonstration. You can access this code from Github at the following link:

<https://github.com/rebelinux/packer-ubuntu-vsphere-horizon-iso>

The first step would be to clone the Github repository locally. In my case I am using Linux on my main computer but Windows 10/11 can also be used for this example.

To do this we will use the command `git clone "https://github.com/rebelinux/packer-ubuntu-vsphere-horizon-iso"`

#### Step 1: Clone packer-ubuntu-vsphere-horizon-iso repository

```bash
[rebelinux@PC ~]$ git clone "https://github.com/rebelinux/packer-ubuntu-vsphere-horizon-iso"
Cloning into 'packer-ubuntu-vsphere-horizon-iso'...
remote: Enumerating objects: 93, done.
remote: Counting objects: 100% (93/93), done.
remote: Compressing objects: 100% (67/67), done.
remote: Total 93 (delta 51), reused 63 (delta 24), pack-reused 0
Receiving objects: 100% (93/93), 287.93 KiB | 1009.00 KiB/s, done.
Resolving deltas: 100% (51/51), done.
[rebelinux@PC ~]$ ls "packer-ubuntu-vsphere-horizon-iso"
build-2204.ps1  files  README.md  ubuntu.pkr.hcl
build-2204.sh   http   setup      variables.auto.pkrvars.hcl.sample
[rebelinux@PC ~]$
```

After having the repository in our local directory it is necessary to set the variables unique to our VMware vSphere environment.

#### Step 2: Rename the variables file

```bash
[rebelinux@PC ~]$ cd "packer-ubuntu-vsphere-horizon-iso/"
[rebelinux@PC ~]$ mv "variables.auto.pkrvars.hcl.sample" "variables.auto.pkrvars.hcl"
[rebelinux@PC ~]$ 
```

#### Step 3: Modify variables

This section of the file defines the vSphere environment variables:

```bash
# Name or IP of you vCenter Server
vsphere_server          = "vcenter.lab.local"

# vsphere username
vsphere_username        = "administrator@vsphere.local"

# vsphere password
vsphere_password        = "**************"

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

This portion of the file defines basic VM parameters such as credentials and modification scripts:

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

Esta porción del archivo define los parámetros del Ambiente de dominio para adjuntar la VM a Active Directory:

##### Note: For SSO to work, it is required to join the VM to a Active Directory domain

```bash
# NTP Server
ntp_server = "xx.xx.xx.xx"
timezone = "America/Puerto_Rico"

# AD Domain
ad_domain = "PHARMAX.LOCAL"

# AD Domain join password
join_password = "**********"

# AD Domain join username
join_username = "Administrator"
```

To finish this part of the file define the name of the Horizon Agent installation package:

```bash
# Horizon Agent install files
horizon_agent_file = "VMware-horizonagent-linux-x86_64-2303-8.9.0-21434177.tar.gz"


# Uncommnet this if using vcenter as web server
// horizon_agent_datastore = "HDD-VM-ISO-LOW-PERF"
// horizon_agent_path = "/ISO/VMWARE/Horizon/"
```

#### Step 4: Copy Horizon agent file locally

In this step it is mandatory to copy the Horizon Agent installation file to the `files` folder.

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

#### Step 5: Initialize packer

Then it is required to initialize packer from the command line and validate the configuration.

```bash
[rebelinux@rebelpc]$ packer init .
Installed plugin github.com/hashicorp/vsphere v1.1.2 in ".config/packer/plugins/github.com/hashicorp/vsphere/packer-plugin-vsphere_v1.1.2_x5.0_linux_amd64"
[rebelinux@rebelpc]$ packer validate .
"The configuration is valid."
[rebelinux@rebelpc]$ 
```

#### Step 6: Create virtual machine template

Finally, to start generating the template for the Ubuntu VM it is necessary to execute the `packer build .` command. In the case of this `packer` template the `build-2204.sh` file was created to make the process more simple to execute.

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

#### Example: VM created in vCenter

![Text](/img/2023/horizon-ubuntu-vdi-2404/packer_vm_info.webp#center)

#### Example: Automated Ubuntu Installation Process

![Text](/img/2023/horizon-ubuntu-vdi-2404/packer_ubuntu.webp#center)

#### Example: VM image finalized

![Text](/img/2023/horizon-ubuntu-vdi-2404/packer_vm_status.webp#center)

Now let's move on to test the Ubuntu Linux 22.04 VM image using a VMware Horizon Pool already created for this purpose.

![Text](/img/2023/horizon-ubuntu-vdi-2404/horizon-packer-ubuntu-image.webp#center)

#### Example: VMware Horizon Linux Pool

![Text](/img/2023/horizon-ubuntu-vdi-2404/horizon-vm-ic-status.webp#center)

#### Video: Ubuntu Image Test

{{< youtube fzCuAddi-BM >}}

In the second part of this series of articles I will be explaining how to modify this `packer` template and how to edit the scripts that configure the image.

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/F1F8DEV80)

![Text](/img/hasta-luego-5937ba.webp#center)
