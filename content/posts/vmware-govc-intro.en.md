---
title: 'Govc: vSphere CLI built on top of govmomi, an introduction'
date: '2022-08-12T10:50:00-04:00'
tags:
    - vSphere
    - VMware
    - Golang
---

Hello everyone,

This time I will be talking about how to use `Govc`, a CLI alternative developed in the [GO](https://go.dev/) programming language, this CLI is designed to be an easy-to-use alternative to the Web GUI and very suitable for automation tasks. This tool is based on `govmomi`, which is a library for GO used to interact with the VMware vSphere Application Programming Interface (API).

#### Note: vSphere versions that have reached end of support may work, but are not officially supported.

Here is the link so you can see the features of this tool.

- <https://github.com/vmware/govmomi/blob/main/govc/README.md>

This tool can be installed in many ways but for this tutorial I will use the compiled version. To use another installation method you can refer to the documentation [Govc](https://github.com/vmware/govmomi/tree/main/govc#installation).

The first step would be to download the tool from the [Github portal](https://github.com/vmware/govmomi/releases). In my case I will use the x86_64 version which is the one that corresponds to my Linux system, but this tool provides support for the majority of the most commonly used Operating Systems.

![Text](/img/2022/vmware-govc-intro/govc_download_x86.webp#center)

After downloading the package it is needed to unzip the contents in order to access the compiled file.

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
CHANGELOG.md  `govc`  govc_Linux_x86_64.tar.gz  LICENSE.txt  README.md
[bla@blabla govc]$ 
```

In the folder unzipped there is the `govc` file which is the main binary of the tool. To test with `govc` I will connect to my HomeLab's vCenter which uses the FQDN address `vcenter-01v.pharmax.local`.

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

#### Note: This tool assumes that the HTTPS certificate can be validated with its trust chain. If the certificate is "Self Sign" it is required to set the GOVC_INSECURE=1 variable

In this article I showed you an introduction about the `Govc` tool in the next articles I will be going into detail on how to use this tool to streamline the automation processes in our vSphere infrastructure.

#### Hasta Luego Amigos!
