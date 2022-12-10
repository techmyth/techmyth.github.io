---
title: "Installing Minio on Centos Linux 8"
date: 2021-05-14T08:39:31-04:00
author: 'Jonathan Colon Feliciano'
draft: false
tags:
  - "Veeam"
  - "Minio"
  - "Centos"
---

### The Problem

In the last few months I have been studying for the **“VEEAM VMCE Arquitect”** certification and one of the requirements is to implement the **“Scale Out Backup Repository”** option. A simple way to achieve this is by using the Minio service.

### About Minio

Minio is an open source object storage service compatible with the Amazon S3 cloud storage service. Applications that have been configured to communicate with Amazon S3 can also be configured to communicate with Minio, allowing Minio to be a viable alternative to S3.

### Installation

#### [Adding the minio user]

```bash
[root@VEEAM-MINIO ~]# useradd -r minio-user -s /sbin/nologin
```

#### [Creating the configuration file]

```bash
[root@VEEAM-MINIO ~]# cat <<EOT >> /etc/default/minio

# Volume to be used for MinIO server.

MINIO_VOLUMES="/usr/local/share/minio/"

# Use if you want to run MinIO on a custom port.

MINIO_OPTS="-C /etc/minio --address <your_server_ip>:9000"

# User for the server.

MINIO_ACCESS_KEY="<change me>"

# Password for the server

MINIO_SECRET_KEY="<change me>"

# Root user for the server. 

MINIO_ROOT_USER="<change me>"

# Root secret for the server. 

MINIO_ROOT_PASSWORD="<change me>"

EOT
```

#### [Downloading the application]

```bash
[root@VEEAM-MINIO ~]# wget https://dl.min.io/server/minio/release/linux-amd64/minio

[root@VEEAM-MINIO ~]# chmod +x minio #assigning privileges to run

[root@VEEAM-MINIO ~]# mv minio /usr/local/bin #moving executable to directory

[root@VEEAM-MINIO ~]# chown minio-user:minio-user /usr/local/bin/minio #changing user permissions
```

#### [Creating directories and assigning permissions]

```bash
[root@VEEAM-MINIO ~]# mkdir /usr/local/share/minio

[root@VEEAM-MINIO ~]# chmod 755 /usr/local/share/minio

[root@VEEAM-MINIO ~]# chown minio-user:minio-user /usr/local/share/minio

[root@VEEAM-MINIO ~]# mkdir /etc/minio

[root@VEEAM-MINIO ~]# chown minio-user:minio-user /etc/minio
```

#### [Downloading and activating Minio service]

```text
[root@VEEAM-MINIO ~]# wget https://raw.githubusercontent.com/minio/minio-service/master/linux-systemd/minio.service -O /etc/systemd/system/minio.service

[root@VEEAM-MINIO ~]# systemctl daemon-reload

[root@VEEAM-MINIO ~]# systemctl enable minio
```

#### [Initializing and verifying the service]

```text
[root@VEEAM-MINIO ~]# systemctl start minio

[root@VEEAM-MINIO ~]# systemctl status minio
```

![Text](/img/2021-02-23_13-16-1024x491.webp#center)

#### [Allowing traffic through the local firewall]

```bash
[root@VEEAM-MINIO ~]# firewall-cmd --zone=public --add-port=9000/tcp --permanent

[root@VEEAM-MINIO ~]# firewall-cmd --reload
```

![Text](/img/minio-nmap-login.webp#center)

### Hasta Luego Amigos!
