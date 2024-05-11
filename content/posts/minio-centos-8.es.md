---
title: "Instalando Minio en Centos Linux 8"
date: 2021-05-14T08:39:31-04:00
author: 'Jonathan Colon Feliciano'
draft: false
tags:
  - "Veeam"
  - "Minio"
  - "Centos"
---

### El Problema

En estos últimos meses he estado estudiando para la certificación de `VEEAM VMCE Arquitect` y uno de los requisitos es implementar la opción de `Scale Out Backup Repository`. Una forma sencilla de lograr esto es utilizando el servicio de Minio.

### Sobre Minio

Minio es un servicio de almacenamiento de objetos de código abierto compatible con el servicio de almacenamiento en la nube de Amazon S3. Las aplicaciones que se han configurado para comunicarse con Amazon S3 también se pueden configurar para comunicarse con Minio, lo que permite que Minio sea una alternativa viable a S3.

### Instalación

#### [Añadiendo el usuario de minio]

```bash
[root@VEEAM-MINIO ~]# useradd -r minio-user -s /sbin/nologin
```

#### [Creando el archivo de configuración]

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

#### [Descargando la aplicación]

```bash
[root@VEEAM-MINIO ~]# wget https://dl.min.io/server/minio/release/linux-amd64/minio

[root@VEEAM-MINIO ~]# chmod +x minio #assigning privileges to run

[root@VEEAM-MINIO ~]# mv minio /usr/local/bin #moving executable to directory

[root@VEEAM-MINIO ~]# chown minio-user:minio-user /usr/local/bin/minio #changing user permissions
```

#### [Creando directorios y asignando permisos]

```bash
[root@VEEAM-MINIO ~]# mkdir /usr/local/share/minio

[root@VEEAM-MINIO ~]# chmod 755 /usr/local/share/minio

[root@VEEAM-MINIO ~]# chown minio-user:minio-user /usr/local/share/minio

[root@VEEAM-MINIO ~]# mkdir /etc/minio

[root@VEEAM-MINIO ~]# chown minio-user:minio-user /etc/minio
```

#### [Descargando y activando servicio de Minio]

```text
[root@VEEAM-MINIO ~]# wget https://raw.githubusercontent.com/minio/minio-service/master/linux-systemd/minio.service -O /etc/systemd/system/minio.service

[root@VEEAM-MINIO ~]# systemctl daemon-reload

[root@VEEAM-MINIO ~]# systemctl enable minio
```

#### [Inicializando y verificando el servicio]

```text
[root@VEEAM-MINIO ~]# systemctl start minio

[root@VEEAM-MINIO ~]# systemctl status minio
```

![Text](/img/2021-02-23_13-16-1024x491.webp#center)

#### [Permitiendo tráfico en el `firewall` local]

```bash
[root@VEEAM-MINIO ~]# firewall-cmd --zone=public --add-port=9000/tcp --permanent

[root@VEEAM-MINIO ~]# firewall-cmd --reload
```

![Text](/img/minio-nmap-login.webp#center)

### Hasta Luego Amigos!
