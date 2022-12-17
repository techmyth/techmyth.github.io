---
title: 'NetApp Ontap Mediator Installation and Configuration'
date: '2021-07-11T05:00:00-04:00'
tags:
    - Centos
    - Linux
    - NetApp
---

Hello everyone,

Today I will be talking a bit about how to install and configure the “Ontap Mediator” application that is used as an alternate way to validate the health status of a cluster collection. To set up the role of this application I will use as reference the NetApp portal documentation:

> ONTAP Mediator provides an alternate health path to the peer cluster, with the intercluster LIFs providing the other health path. With the Mediator’s health information, clusters can differentiate between intercluster LIF failure and site failure. When the site goes down, Mediator passes on the health information to the peer cluster on demand, facilitating the peer cluster to fail over. With the Mediator-provided information and the intercluster LIF health check information, ONTAP determines whether to perform an auto failover, if it is failover incapable, continue or stop.

[Role of ONTAP Mediator](https://docs.netapp.com/us-en/ontap/smbc/smbc_intro_role_of_mediator.html)

This application can be used in “MetroCluster” scenarios as well as with “SnapMirror Business Continuity” (SM-BC) technology. As of ONTAP 9.8, SnapMirror Business Continuity (SM-BC) can be used to protect applications with LUNs, allowing applications to migrate transparently, ensuring business continuity in the event of a disaster. SM-BC uses “SnapMirror Synchronous” technology that allows data to be replicated to the target as soon as it is written to the source volume.

![Text](/img/rtaImage-1.webp#center)

In this lab I will show you the Mediator application with the purpose of being able to perform in the future a lab on SM-BC in a VMware environment. The following image shows the role of Ontap Mediator within the SM-BC technology architecture.

![Text](/img/Ontap-Mediator.webp#center)

As you can see the “Mediator” is constantly evaluating the Datacenter status to identify possible failures and to be able to react by migrating access to the volumes to the Datacenter that is up and running. It may be useful to understand some of the basics of SM-BC recovery and restoration.

**Planned recovery:**

A manual operation to change the access roles to volumes in an SM-BC relationship. The primary becomes the secondary and the secondary becomes the primary. The ALUA status report is also modified according to the status of the relationship.

**Automatic unplanned recovery (AUFO):**

An automatic operation to perform a failover to the mirror copy. The operation requires the assistance of the Ontap Mediator to detect that the primary copy is not available.

Here are the requirements to install the application.

#### Requirements

To validate the complete list of requirements you can visit the documentation of [“Ontap Mediator”](https://docs.netapp.com/us-en/ontap-metrocluster/install-ip/task_install_configure_mediator.html)

![Text](/img/2021-06-26_20-27.webp#center)

For this lab I am going to use Red Hat Enterprise Linux 8.1 running on a vSphere VM. The first thing to do is to download the application installation package. This is done by accessing the NetApp support portal as shown in the following image.

**Link to Ontap Mediator:**

<https://mysupport.netapp.com/site/products/all/details/ontap-mediator/downloads-tab>

![Text](/img/2021-06-29_21-57-1536x1344-1.webp#center)

After downloading the installation package, copy the **“ONTAP-MEDIATOR-1.3”** file to the server to be used for this purpose. Then proceed to change the installation file to executable mode with the command **chmod +x**.

```bash
[root@NTAPMED-01V ~]# ls
anaconda-ks.cfg  ONTAP-MEDIATOR-1.3
[root@NTAPMED-01V ~]# chmod +x ONTAP-MEDIATOR-1.3 
[root@NTAPMED-01V ~]#
```

Next, proceed to install the application dependencies with the **yum install** command as shown below.

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

Once all dependencies are installed, you can start running the application installation file. To do this use the command **./ONTAP-MEDIATOR-1.3**.

Note: This command must be executed in the location where the installation file was stored.

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

The installer will ask several questions about the password of the users used by the ONTAP Mediator service and the TCP ports that will be opened in the local “Firewall” of the server. Once everything is properly specified the installer will validate that all the application prerequisites are installed.

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

After installing the application it is important to validate that the services of the Ontap Mediator are activated and functional. To validate the services use the command **systemctl status ontap\_mediator mediator-scst**.

```bash
[root@NTAPMED-01V ~]# systemctl status ontap_mediator mediator-scst
● ontap_mediator.service - ONTAP Mediator
   Loaded: loaded (/etc/systemd/system/ontap_mediator.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2021-07-09 14:21:31 AST; 11min ago
  Process: 1296 ExecStop=/bin/kill -s INT $MAINPID (code=exited, status=0/SUCCESS)
 Main PID: 1298 (uwsgi)
   Status: "uWSGI is ready"
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

Additionally, it is important to ensure that the services are using the correct tcp ports. With the command **&lt;netstat -anlt | grep -E ‘3260|31784’&gt;** you can validate that ports 3260 and 31784 are in **“LISTEN”** mode.

```bash
[root@NTAPMED-01V ~]# netstat -anlt | grep -E '3260|31784'
tcp        0      0 0.0.0.0:3260            0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:31784           0.0.0.0:*               LISTEN     
tcp6       0      0 :::3260                 :::*                    LISTEN     
[root@NTAPMED-01V ~]# 
```

With the command **firewall-cmd –list-all** you can validate that the rules for ports 31784/tcp and 3260/tcp are properly configured in the server’s local firewall.

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

Once the installation process has been successfully completed, add the Ontap Mediator to the configuration of the clusters where you have selected to use the “SnapMirror Business Continuity” (SM-BC) technology. To add the configuration, go to **[Protection] => [Overview] => [Mediator] => [Configure]**. Then you have to add the configuration as shown in the following images. It is important to mention that the certificate to be added in this configuration is the one of the CA located in:

* /opt/netapp/lib/ontap mediator/ontap mediator/server config/ca.crt

Note: It is important to mention that for this configuration to work there must be a “cluster peer” and “vserver peer” relationship previously established.

![Text](/img/mediator_1.webp#center)

![Text](/img/mediator_2.webp#center)

![Text](/img/mediator_3.webp#center)

![Text](/img/mediator_4.webp#center)

![Text](/img/mediator_5.webp#center)

Through the Ontap console you can also validate that the Ontap Mediator configuration is working correctly. With the **snapmirror mediator show** command you can validate that the Connection Status is “connected” and the Quorum Status is “true”.

Note: This command must be used in both clusters to validate that the connection is correctly established.

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

Here is how to add the Ontap Mediator to the cluster through Ontap’s console.

#### **Ontap Mediator CLI Setup**

With the **snapmirror mediator add** command you can add the Ontap Mediator with the IP address **192.168.6.16** to the **Onprem-HQ** cluster. It is important to mention that for this configuration to work there must be a “Cluster peer” and “Vserver peer” relationship previously established.

```text
OnPrem-HQ::> snapmirror mediator add -mediator-address 192.168.7.167 -peer-cluster OnPrem-DR -username mediatoradmin 

Notice: Enter the mediator password.

Enter the password: XXXXX
Enter the password again: XXXXX

Info: [Job: 171] 'mediator add' job queued 

OnPrem-HQ::> 
```

With the **snapmirror mediator show** command you can validate that the Connection Status is “connected” and the Quorum Status is set to “true”.

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

Additionally I show you how to replace the SSL certificate of the Ontap Mediator service with one generated from a Microsoft Certificate Authority.

#### **Optional SSL Certificate Replacement**

**Step 1:** Generate a configuration file to create the Certificate Signing Request (CSR). In this step it is important to set the CN and DNS with the fully qualified domain name (FQDN) of the server name. In my case the server name is NTAPMED-01V.

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

**Step 2:** Use the openssl command to generate the CSR file that will be used as a template to create the certificate that the Ontap Mediator service will use.

Note: If the openssl command is not available on your system you can use the **yum install openssl** command to install the necessary packages

```text
[root@NTAPMED-01V ~]# openssl req -new -out ntapmed.csr -newkey rsa:2048 -nodes -sha256 -keyout ntapmed.key -config req.conf
```

Once the openssl command has finished, two files will be created, the **ntapmed.csr** is the template that will be used to generate the certificate and the **ntapmed.key** is the private key.

```text
[root@NTAPMED-01V ~]# ls -al ntapmed.*
-rw-r--r-- 1 root      root      1123 Jul  9 16:53 ntapmed.csr #Certificate Signing Request
-rw-r--r-- 1 rebelinux rebelinux 1704 Jul  9 16:53 ntapmed.key #Private Key
[root@rebelpc rebelinux]# 
```

**Step 3:** Access Microsoft’s Certificate Authority server and use the **certreq.exe** command to generate the certificate using the **ntapmed.csr** file as template.

```text
C:\>certreq.exe -submit -attrib "CertificateTemplate:WebServer" ntapmed.csr ntapmed.cer
```

![Text](/img/2021-07-09_17-06.webp#center)

Once the process is completed, a file will be created with the name ntapmed.cer that is used for the Ontap Mediator service.

**Step 4:** To replace the SSL certificate it is also necessary to change the public certificate of the CA. To obtain this certificate from the CA use the command **certutil -ca.cert ca.cert** which will produce the certificate in the **ca.cer** file.

```text
C:\>certutil -ca.cert ca.cer
```

![Text](/img/2021-07-09_20-16.webp#center)

Once this process is completed simply copy all the files (ca.cer, ntapmed.cer and ntapmed.key) to the Ontap Mediator server.

**Step 5:** Move to the **/opt/netapp/lib/ontap\_mediator/ontap\_mediator/server\_config/** folder and modify the certificate files as shown below.

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

After making the changes, it is necessary to restart the services using the command **systemctl restart ontap\_mediator mediator-scst**

```text
[root@NTAPMED-01V server_config]# systemctl restart ontap_mediator mediator-scst
[root@NTAPMED-01V server_config]# systemctl status ontap_mediator mediator-scst
● ontap_mediator.service - ONTAP Mediator
   Loaded: loaded (/etc/systemd/system/ontap_mediator.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2021-07-09 20:31:48 AST; 8s ago
  Process: 22222 ExecStop=/bin/kill -s INT $MAINPID (code=exited, status=0/SUCCESS)
 Main PID: 22232 (uwsgi)
   Status: "uWSGI is ready"
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

### Summary

In this lab I showed you how to install and configure Ontap Mediator. In the future I will use this service to do a lab on “SnapMirror Business Continuity” (SM-BC) together with VMware. I hope you liked this lab. If you have any doubts or questions about it, leave them in the comments. Regards.
