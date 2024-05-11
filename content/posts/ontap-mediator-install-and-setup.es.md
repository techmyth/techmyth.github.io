---
title: 'Instalación y configuración de Ontap Mediator de NetApp'
date: '2021-07-11T05:00:00-04:00'
tags:
    - Centos
    - Linux
    - NetApp
---

Hol@ a todos,

El día de hoy estaré hablando un poco de como instalar y configurar la aplicación de `Ontap Mediator` que es utilizado como una vía alterna para validar el estado de salud de un conjunto de clúster. Para establecer el rol de esta aplicación utilizaré como referencia la documentación del portal de NetApp:

> ONTAP Mediator provides an alternate health path to the peer cluster, with the intercluster LIFs providing the other health path. With the Mediator’s health information, clusters can differentiate between intercluster LIF failure and site failure. When the site goes down, Mediator passes on the health information to the peer cluster on demand, facilitating the peer cluster to fail over. With the Mediator-provided information and the intercluster LIF health check information, ONTAP determines whether to perform an auto failover, if it is failover incapable, continue or stop.

[Roles de ONTAP Mediator](https://docs.netapp.com/us-en/ontap/smbc/smbc_intro_role_of_mediator.html)

Esta aplicación puede ser utilizada en escenarios de `MetroCluster` como también con la tecnología de `SnapMirror Business Continuity` (SM-BC). A partir de ONTAP 9.8, `SnapMirror Business Continuity` (SM-BC) puedes ser utilizado para proteger las aplicaciones con LUNs, permitiendo que las aplicaciones puedan migrar de forma transparente, asegurando la continuidad del negocio en caso de desastre. SM-BC utiliza la tecnología de `SnapMirror Synchronous` que permite replicar los datos hacia el destino tan pronto como se escriben en el volumen de origen.

![Text](/img/rtaImage-1.webp#center)

En este laboratorio instalaremos la aplicación Mediator con el propósito de en el futuro poder realizar un laboratorio sobre SM-BC en un ambiente de VMware. La siguiente imagen muestra el rol que posee el Ontap Mediator dentro del diseño de la tecnología de SM-BC.

![Text](/img/Ontap-Mediator.webp#center)

Como pueden ver el `Mediator` esta constantemente evaluando el estado del Datacenter para identificar posibles fallas y poder reaccionar migrando el acceso a los volumenes al Datacenter que este en función. Puede ser útil entender algunos de los conceptos básicos de recuperación y restauración de SM-BC.

#### Recuperación planificada:

Una operación manual para cambiar los roles de acceso a los volúmenes en una relación SM-BC. El primario pasa a ser el secundario y el secundario pasa a ser el primario. El reporte del estado de ALUA también es modificado según el estado de la relación.

#### Recuperación automática no planificada (AUFO):

Una operación automática para realizar una conmutación por error a la copia espejo. La operación requiere la asistencia del Ontap Mediator para detectar que la copia primaria no está disponible.

A continuación les incluyo los requisitos para instalar la aplicación.

#### Requerimientos

Para validar la lista completa de requisitos puede visitar la documentación de [`Ontap Mediator`](https://docs.netapp.com/us-en/ontap-metrocluster/install-ip/task_install_configure_mediator.html)

![Text](/img/2021-06-26_20-27.webp#center)

Para este laboratorio estaré utilizando Red Hat Enterprise Linux 8.1 corriendo en en una VM de vSphere. Lo primero que debemos hacer es descargar el paquete de instalación de la aplicación. Para esto accedemos el portal de soporte de NetApp según se muestra en la próxima imagen.

#### Enlace de Ontap Mediator:

<https://mysupport.netapp.com/site/products/all/details/ontap-mediator/downloads-tab>

![Text](/img/2021-06-29_21-57-1536x1344-1.webp#center)

Una vez descargado el paquete de instalación copiamos el archivo `ONTAP-MEDIATOR-1.3` al servidor que utilizaremos para este propósito. Luego procedemos a cambiar el archivo de instalación a modo ejecutable con el comando `chmod +x ONTAP-MEDIATOR-1.3`.

```bash
[root@NTAPMED-01V ~]# ls
anaconda-ks.cfg  ONTAP-MEDIATOR-1.3
[root@NTAPMED-01V ~]# chmod +x ONTAP-MEDIATOR-1.3 
[root@NTAPMED-01V ~]#
```

Ahora procedemos a instalar las dependencias de la aplicación con el comando `yum install` según se muestra a continuación.

```bash
[root@NTAPMED-01V ~]# yum install openssl openssl-devel kernel-devel gcc libselinux-utils make redhat-lsb-core patch bzip2 python36 python36-devel perl-Data-Dumper perl-ExtUtils-MakeMaker python3-pip elfutils-libelf-devel policycoreutils-python-utils -y
Last metadata expiration check: 0:13:59 ago on Tue 29 Jun 2021 10:01:36 PM AST.
Package openssl-1:1.1.1g-15.el8_3.x86_64 is already installed.
Package libselinux-utils-2.9-5.el8.x86_64 is already installed.
Dependencies resolved.
...............
Installed:

Really long Output                                                              

Complete!
[root@NTAPMED-01V ~]#

```

Una vez hayamos instalado las dependencia podemos comenzar a ejecutar el archivo de instalación de la aplicación. Para este propósito utilizaremos el comando `./ONTAP-MEDIATOR-1.3`.

Nota: Este comando debe ejecutarse en el lugar donde este guardado el archivo de instalación.

```bash
[root@NTAPMED-01V ~]# ./ONTAP-MEDIATOR-1.3 

ONTAP Mediator: Self Extracting Installer

ONTAP Mediator requires two user accounts. One for the service (netapp), and one for use by ONTAP to the mediator API (mediatoradmin).
Would you like to use the default account names: netapp + mediatoradmin? (Y(es)/n(o)): Yes
Enter ONTAP Mediator user account (mediatoradmin) password: XXXXXX 

Re-Enter ONTAP Mediator user account (mediatoradmin) password: XXXXX

Checking if SELinux is in enforcing mode
SELinux is set to Enforcing. ONTAP Mediator server requires modifying the SELinux context of the file
/opt/netapp/lib/ontap_mediator/pyenv/bin/uwsgi from type 'lib_t' to 'bin_t'.
This is neccessary to start the ONTAP Mediator service while SELinux is set to Enforcing.
Allow SELinux context change?  Y(es)/n(o): Yes
The installer will change the SELinux context type of
/opt/netapp/lib/ontap_mediator/pyenv/bin/uwsgi from type 'lib_t' to 'bin_t'.




Checking for default Linux firewall
Linux firewall is running. Open ports 31784 and 3260? Y(es)/n(o): Yes
success
success
success


###############################################################
Preparing for installation of ONTAP Mediator packages.


Do you wish to continue? Y(es)/n(o): 
```

El instalador nos realizara varias preguntas sobre la contraseña de los usuarios utilizados por el servicio de ONTAP Mediator y los puertos TCP que se abrirán en el `Firewall` local del servidor. Una vez todo este debidamente especificado el instalador validara que todos los pre-requisitos de la aplicación estén instalados.

```bash
Do you wish to continue? Y(es)/n(o): Y


+ Installing required packages.


Updating Subscription Management repositories.

Really long Output                                                              

Dependencies resolved.
Nothing to do.
Complete!
OS package installations finished
+ Installing ONTAP Mediator. (Log: /tmp/ontap_mediator.7atkl8/ontap-mediator/install_20210709162016.log)
    This step will take several minutes. Use the log file to view progress.
#includedir /etc/sudoers.d
Sudo include verified
ONTAP Mediator logging enabled
+ Install successful. (Moving log to /opt/netapp/lib/ontap_mediator/log/install_20210709162016.log)
+ Note: ONTAP Mediator uses a kernel module compiled specifically for the current
        system OS. Using 'yum update' to upgrade the kernel may cause a service
        interruption.
    For more information, see /opt/netapp/lib/ontap_mediator/README
[root@NTAPMED-01V ~]#
```

Una vez instalada la aplicación es importante validar que los servicio del Ontap Mediator estén activados y funcionales. Para validar los servicio utilizamos el comando `systemctl status ontap_mediator mediator-scst`.

```bash
[root@NTAPMED-01V ~]# systemctl status ontap_mediator mediator-scst
● ontap_mediator.service - ONTAP Mediator
   Loaded: loaded (/etc/systemd/system/ontap_mediator.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2021-07-09 14:21:31 AST; 11min ago
  Process: 1296 ExecStop=/bin/kill -s INT $MAINPID (code=exited, status=0/SUCCESS)
 Main PID: 1298 (uwsgi)
   Status: `uWSGI is ready`
    Tasks: 3 (limit: 23832)
   Memory: 61.4M
 Started ONTAP Mediator.

● mediator-scst.service
   Loaded: loaded (/etc/systemd/system/mediator-scst.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2021-07-09 14:21:30 AST; 11min ago
  Process: 1164 ExecStart=/etc/init.d/scst start (code=exited, status=0/SUCCESS)
 Main PID: 1250 (iscsi-scstd)
    Tasks: 1 (limit: 23832)
   Memory: 3.3M
 Started mediator-scst.service.
[root@NTAPMED-01V ~]# 
```

Adicionalmente es importante validar que los servicios estén utilizando los puertos correctos. Con el comando `netstat -anlt | grep -E ‘3260|31784’` podemos validar que los puertos 3260 y 31784 este en modo `LISTEN`.

```bash
[root@NTAPMED-01V ~]# netstat -anlt | grep -E '3260|31784'
tcp        0      0 0.0.0.0:3260            0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:31784           0.0.0.0:*               LISTEN     
tcp6       0      0 :::3260                 :::*                    LISTEN     
[root@NTAPMED-01V ~]# 
```

Con el comando `firewall-cmd –list-all` podemos validar que las reglas para los puertos 31784/tcp y 3260/tcp estén configurados en el `Firewall` local del servidor.

```bash
[root@NTAPMED-01V ~]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens192
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 31784/tcp 3260/tcp
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 

[root@NTAPMED-01V ~]# 
```

Luego de culminado el proceso de instalación pasamos a añadir el Ontap Mediator a la configuración de los cluster donde deseamos utilizar la tecnología de `SnapMirror Business Continuity` (SM-BC). Para añadir la configuración es necesario ir a `[Protection\]` => `[Overview]` => `[Mediator]` => `[Configure]`. Luego hay que añadir la configuración según se muestra en la siguientes imágenes. Es importante mencionar que el certificado que se añade en esta configuración es el del CA que se encuentra en:

* /opt/netapp/lib/ontap mediator/ontap mediator/server config/ca.crt

Nota: Es importante mencionar que para que esta configuración funciones debe de existir una relación de `cluster peer` y `vserver peer` previamente establecida.

![Text](/img/mediator_1.webp#center)

![Text](/img/mediator_2.webp#center)

![Text](/img/mediator_3.webp#center)

![Text](/img/mediator_4.webp#center)

![Text](/img/mediator_5.webp#center)

Por consola también podemos validar que la configuración del Ontap Mediator este funcionando correctamente. Con el comando `snapmirror mediator show` podemos validar que el estado de la conexión esta `connected` y el `Quorum Status` esta en `true`.

Nota: Este comando debe de ser utilizado en ambos cluster para validar que la conexión esta correctamente establecida.

```text
OnPrem-HQ::> snapmirror mediator show                    
Mediator Address Peer Cluster     Connection Status Quorum Status
---------------- ---------------- ----------------- -------------
192.168.6.16     OnPrem-DR       connected         true

OnPrem-HQ::*> 
```

```text
OnPrem-DR::> snapmirror mediator show
Mediator Address Peer Cluster     Connection Status Quorum Status
---------------- ---------------- ----------------- -------------
192.168.6.16     OnPrem-HQ       connected       true

OnPrem-DR::> 
```

A continuación les incluyo como añadir al cluster el Ontap Mediator utilizando el modo de consola de Ontap.

#### Ontap Mediator CLI Setup

Con el comando `snapmirror mediator add` añadimos el Ontap Mediator con la dirección IP de 192.168.6.16 al cluster `Onprem-HQ`. Es importante mencionar que para que esta configuración funciones debe de existir una relación de `Cluster peer` y `Vserver peer` previamente establecida.

```text
OnPrem-HQ::> snapmirror mediator add -mediator-address 192.168.7.167 -peer-cluster OnPrem-DR -username mediatoradmin 

Notice: Enter the mediator password.

Enter the password: XXXXX
Enter the password again: XXXXX

Info: [Job: 171] 'mediator add' job queued 

OnPrem-HQ::> 
```

Con el comando `snapmirror mediator show` podemos validar que el estado de la conexión esta `connected` y el `Quorum Status` esta en `true`.

```text
OnPrem-HQ::> snapmirror mediator show                    
Mediator Address Peer Cluster     Connection Status Quorum Status
---------------- ---------------- ----------------- -------------
192.168.6.16     OnPrem-DR       connected         true

OnPrem-HQ::*> 
```

```text
OnPrem-DR::> snapmirror mediator show
Mediator Address Peer Cluster     Connection Status Quorum Status
---------------- ---------------- ----------------- -------------
192.168.6.16     OnPrem-HQ       connected       true

OnPrem-DR::> 
```

Adicionalmente les muestro como remplazar el certificado SSL del servicio de Ontap Mediator por uno generado de un `Certificate Authority` de Microsoft.

#### Opcional - Remplazo Certificado SSL

`Paso 1:` Generar un archivo de configuración para crear el `Certificate Signing Request` (CSR). En este paso es importante establecer el CN y DNS con el `fully qualified domain name` (FQDN) del nombre del servidor. En mi caso el nombre del servidor es `NTAPMED-01V`

```text
[root@NTAPMED-01V ~]# nano -w req.conf 
[req]
distinguished_name = req_distinguished_name
req_extensions = v3_req
prompt = no
[req_distinguished_name]
C = US
ST = PR
L = SJ
O = Zen PR Solutions
OU = IT
CN = NTAPMED-01V
[v3_req]
keyUsage = keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names
[alt_names]
DNS.1 = NTAPMED-01V.zenprsolutions.local
```

`Paso 2:` Utilizar el comando `openssl` para generar el archivo CSR que utilizaremos como plantilla para crear el certificado que utilizaremos para el servicio de Ontap Mediator.

Nota: Si el comando `openssl` no esta disponible en el sistema puede utilizar el comando `yum install openssl` para instalar los paquetes necesarios.

```text
[root@NTAPMED-01V ~]# openssl req -new -out ntapmed.csr -newkey rsa:2048 -nodes -sha256 -keyout ntapmed.key -config req.conf
```

Una vez ejecutado el comando openssl se crearan dos archivos el `ntapmed.csr` es la plantilla que utilizaremos para generar el certificado y el `ntapmed.key` es la llave privada.

```text
[root@NTAPMED-01V ~]# ls -al ntapmed.*
-rw-r--r-- 1 root      root      1123 Jul  9 16:53 ntapmed.csr #Certificate Signing Request
-rw-r--r-- 1 rebelinux rebelinux 1704 Jul  9 16:53 ntapmed.key #Private Key
[root@rebelpc rebelinux]# 
```

`Paso 3:` Acceder al servidor de `Certificate Authority` de Microsoft y utilizar el comando `certreq.exe` para generar el certificado utilizando el archivo `ntapmed.csr` como plantilla

```text
C:\>certreq.exe -submit -attrib `CertificateTemplate:WebServer` ntapmed.csr ntapmed.cer
```

![Text](/img/2021-07-09_17-06.webp#center)

Una vez culmine el proceso se creara un archivo con el nombre de `ntapmed.cer` que utilizaremos para el servicio de Ontap Mediator.

`Paso 4:` Para remplazar el certificado SSL es necesario también cambiar el certificado publico del CA . Para obtener este certificado del CA utilizamos el comando `certutil -ca.cert ca.cer` que nos producirá el certificado en el archivo ca.cer.

```text
C:\>certutil -ca.cert ca.cer
```

![Text](/img/2021-07-09_20-16.webp#center)

Una vez culminado este proceso copiamos todos los archivos (ca.cer, ntapmed.cer y ntapmed.key) al servidor de Ontap Mediator.

`Paso 5:` Movernos a la carpeta `/opt/netapp/lib/ontap_mediator/ontap_mediator/server_config/` y modificar los archivos de los certificados segun se muestra a continuacion.

```text
[root@NTAPMED-01V ~]# cd /opt/netapp/lib/ontap_mediator/ontap_mediator/server_config/
[root@NTAPMED-01V server_config]# ls
ca.crt  ca.srl            config.pyc    logging.conf.yaml  ontap_mediator.config.yaml     ontap_mediator_schema.yaml  ontap_mediator_server.csr  ontap_mediator.user_config.yaml
ca.key  config_migration  __init__.pyc  netapp_sudoers     ontap_mediator.constants.yaml  ontap_mediator_server.crt   ontap_mediator_server.key
[root@NTAPMED-01V server_config]# cp -R /opt/netapp/lib/ontap_mediator/ontap_mediator/server_config /root/
[root@NTAPMED-01V server_config]#
```

```text
[root@NTAPMED-01V server_config]# nano -w ca.crt
```

![Text](/img/2021-07-09_20-23.webp#center)

```text
[root@NTAPMED-01V server_config]# openssl x509 -noout -serial -in ca.crt 
serial=5D2E25D9AFFDE4904A05D70BEB7ACBD2
[root@NTAPMED-01V server_config]# 
```

![Text](/img/2021-07-11_10-02.webp#center)

```text
[root@NTAPMED-01V server_config]# nano -w ontap_mediator_server.crt
```

![Text](/img/2021-07-09_20-28.webp#center)

```text
[root@NTAPMED-01V server_config]# nano -w ontap_mediator_server.key
```

![Text](/img/2021-07-09_20-30.webp#center)

Una vez realizado los cambios es necesario reiniciar los servicios utilizando el comando `systemctl restart ontap_mediator mediator-scst`

```text
[root@NTAPMED-01V server_config]# systemctl restart ontap_mediator mediator-scst
[root@NTAPMED-01V server_config]# systemctl status ontap_mediator mediator-scst
● ontap_mediator.service - ONTAP Mediator
   Loaded: loaded (/etc/systemd/system/ontap_mediator.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2021-07-09 20:31:48 AST; 8s ago
  Process: 22222 ExecStop=/bin/kill -s INT $MAINPID (code=exited, status=0/SUCCESS)
 Main PID: 22232 (uwsgi)
   Status: `uWSGI is ready`
    Tasks: 3 (limit: 23832)
   Memory: 56.5M

● mediator-scst.service
   Loaded: loaded (/etc/systemd/system/mediator-scst.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2021-07-09 20:31:50 AST; 5s ago
  Process: 22223 ExecStop=/etc/init.d/scst stop (code=exited, status=0/SUCCESS)
  Process: 22309 ExecStart=/etc/init.d/scst start (code=exited, status=0/SUCCESS)
 Main PID: 22389 (iscsi-scstd)
    Tasks: 1 (limit: 23832)
   Memory: 1.0M
```

### Resumen

En este laboratorio instalamos y configuramos el Ontap Mediator que utilizaremos en un futuro para realizar un laboratorio sobre `SnapMirror Business Continuity` (SM-BC) junto con VMware. Espero que este laboratorio les haya gustado. Si tienes dudas o alguna pregunta sobre este laboratorio, déjalo en los comentarios. Saludos.
