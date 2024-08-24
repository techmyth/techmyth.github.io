---
title: "HomeLab: vSphere 7 update 2 QuickBoot with Nested ESXi"
date: 2021-04-07T08:39:31-04:00
author: 'Jonathan Colon Feliciano'
draft: false
tags:
  - "VMware"
  - "HomeLab"

---

In this bog, I will be validating if the "ESXi QuickBoot" feature works in Nested mode i.e. running ESXi in a VM. First you must know exactly how `QuickBoot` works and all the requirements it has. To do this I will use the VMWare portal documentation as a reference:

> Quick Boot is a vSphere feature that speeds up the upgrade process of an ESXi server.  A regular reboot involves a full power cycle that requires firmware and device initialization.  Quick Boot optimizes the reboot path to avoid this, saving considerable time from the upgrade process.

#### [Understanding ESXi Quick Boot Compatibility](https://kb.vmware.com/s/article/52477)

![Text](/img/QuickBoot_WorkFlow.webp#center)

Requirements:

- The manufacturer's platform must be compatible
- All device drivers must be supported

Limitations in vSphere 7.0:

- TPM is disabled

Limitations in vSphere 6.7:

- TPM is disabled
- There are no VMs with passthrough devices configured.
- No vmklinux drivers loaded on ESXi.

The VMware documentation includes a list of server manufacturers that support this technology for reference see the link in the VMware documentation [knowledge Base](https://kb.vmware.com/s/article/52477).

For ESXi 7.0 or newer versions, you can check the `hardware` compatibility here:

- Quick Boot on Dell EMC
- Quick Boot on HPE
- Quick Boot on Cisco
- Quick Boot on Fujitsu
- Quick Boot on Lenovo

In this lab I will test `QuickBoot` using Nested Virtualization. It is important to clarify that this is a test/dev scenario. To use this technology in production environments it is needed to activate the QuickBoot option from `VMware Update Manager` or the renamed `Lifecycle Manager`. If you are interested I leave a video here.

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

The first step to activate `QuickBoot` is to validate the server compatibility. For this it is mandatory to connect via SSH to the ESXi server to execute the command `loadESXCheckCompat.py`. In this case the command confirmed that my platform is compatible.

```text
[root@comp-01a:~] /usr/lib/vmware/loadesx/bin/loadESXCheckCompat.py
This system is compatible with Quick Boot.
[root@comp-01a:~]
```

Once the server is validated as compatible, we can activate the `QuickBoot` function with the command `loadESXEnable -e`.

```text
[root@comp-01a:~] /bin/loadESXEnable -e
INFO: LoadESX Enabled
INFO: Precheck options:
INFO:   All prechecks are enabled.
[root@comp-01a:~]
```

The final step to activate the `QuickBoot` function would be to load the `QuickLaunch` configuration by using the command `loadESX.py`.

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

I am including two videos so you can see the speed of the `QuickBoot` technology when the server is rebooted.

VMware vSphere Normal Boot:
{{< youtube o0rvzKIu4DY >}}

VMware vSphere QuickBoot:
{{< youtube RGEA2pUSaus >}}

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/F1F8DEV80)
