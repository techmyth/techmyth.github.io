---
title: 'HowTo: vSphere Diagnostic Tool'
date: '2022-06-02T10:50:00-04:00'
tags:
    - vSphere
    - VMware
---


This time I am going to show you how to use the **vSphere Diagnostic Tool**, this tool is used to perform diagnostic checks on the VMware vCenter Server Appliance. This tool consists of a python script that executes diagnostic commands to validate the health status of the vCenter service.

#### Note: As a minimum requirement it is required to have a version of vCenter Server 6.5 or higher.

This tool performs the following validations:

- vCenter Basic Info
- Lookup Service Check
- AD Check
- vCenter Certificate Check
- Core File Check
- Disk Check
- vCenter DNS Check
- vCenter NTP Check
- vCenter Port Check
- Root Account Check
- vCenter Services Check
- VCHA Check

Here is the link so you can see the features of this tool.

- <https://flings.vmware.com/vsphere-diagnostic-tool#summary>

The first step would be to download the tool from the portal of **[VMware Fling](https://flings.vmware.com/vsphere-diagnostic-tool)**

![Text](/img/2022/vsphere-diag-tool/VDT-Download.webp#center)

After downloading the file it is required to copy it to the vCenter server. In my case I will use the **scp** command but you can use any tool that supports ssh connections as for example:

- WinSCP
- FileZilla
- Cyberduck
- CrossFTP

Example of how to copy VDT file:

```sh
[rebelinux@rebelpc Downloads]$ scp  ./vdt-v1.1.4.zip root@192.168.5.2:/root

VMware vCenter Server 8.0.0.10100

Type: vCenter Server with an embedded Platform Services Controller

(root@192.168.5.2) Password: 
vdt-v1.1.4.zip                                                                                                                                                                                                   100%  106KB   1.8MB/s   00:00    
[rebelinux@rebelpc Downloads]$ 
```

Then it is needed to connect through **ssh** to the vcenter server. To do this use the ssh command or any terminal emulation tool (Putty, etc...).

```sh
[rebelinux@rebelpc Downloads]$ ssh -l root (IP/FQDN of vCenter Server)

VMware vCenter Server 8.0.0.10100

Type: vCenter Server with an embedded Platform Services Controller

(root@192.168.5.2) Password: 
Last login: Tue Dec 27 20:30:20 2022 from 192.168.70.2
root@vcenter-01v [ ~ ]# 
```

Once connected to the vCenter server it is required to unzip the VDT tool file that is loaded with the command **SCP**.

```sh
root@vcenter-01v [ ~ ]# cd /root/
root@vcenter-01v [ ~ ]# ls
**vdt-v1.1.4.zip**
root@vcenter-01v [ ~ ]# unzip vdt-v1.1.4.zip
Archive:  vdt-v1.1.4.zip
3557676756cffd658fd61aab5a6673269104e83c
   creating: vdt-v1.1.4/
  inflating: vdt-v1.1.4/README.md    
   creating: vdt-v1.1.4/cfg/
  inflating: vdt-v1.1.4/cfg/config_log.ini  
 extracting: vdt-v1.1.4/formerly_pulse.txt  
   creating: vdt-v1.1.4/lib/
  inflating: vdt-v1.1.4/lib/__init__.py  
  inflating: vdt-v1.1.4/lib/cisutils.py  
  inflating: vdt-v1.1.4/lib/lstool_parse.py  
  inflating: vdt-v1.1.4/lib/lstool_scan.py  
  inflating: vdt-v1.1.4/lib/pformatting.py  
  inflating: vdt-v1.1.4/lib/utils.py  
   creating: vdt-v1.1.4/scripts/
 extracting: vdt-v1.1.4/scripts/__init__.py  
  inflating: vdt-v1.1.4/scripts/__vc_info_auth.py  
  inflating: vdt-v1.1.4/scripts/_vc_dns.sh  
  inflating: vdt-v1.1.4/scripts/lsreport.py  
  inflating: vdt-v1.1.4/scripts/vc_ad_check.py  
  inflating: vdt-v1.1.4/scripts/vc_auth_cert_check.py  
  inflating: vdt-v1.1.4/scripts/vc_auth_vmdir_check.py  
  inflating: vdt-v1.1.4/scripts/vc_corefile_check.py  
  inflating: vdt-v1.1.4/scripts/vc_db_check.py  
  inflating: vdt-v1.1.4/scripts/vc_disk_space.py  
  inflating: vdt-v1.1.4/scripts/vc_ntp.sh  
  inflating: vdt-v1.1.4/scripts/vc_ports.py  
  inflating: vdt-v1.1.4/scripts/vc_root_check.py  
  inflating: vdt-v1.1.4/scripts/vc_services.py  
  inflating: vdt-v1.1.4/scripts/vc_syslog_check.py  
  inflating: vdt-v1.1.4/scripts/vc_vcha_check_auth.py  
   creating: vdt-v1.1.4/templates/
  inflating: vdt-v1.1.4/templates/python_template.py  
  inflating: vdt-v1.1.4/vdt.py       
root@vcenter-01v [ ~ ]# ls
**vdt-v1.1.4**  vdt-v1.1.4.zip
root@vcenter-01v [ ~ ]#
```

The next step would be to move to the **vdt-vX.X.X** folder.

```sh
root@vcenter-01v [ ~ ]# cd vdt-v1.1.4
root@vcenter-01v [ ~/vdt-v1.1.4 ]# ls
cfg  formerly_pulse.txt  lib  README.md  scripts  templates  **vdt.py**
root@vcenter-01v [ ~/vdt-v1.1.4 ]# 
```

The important file in this folder is the one called **vdt.py** which is the one to be used to run the diagnostics. Once inside the **vdt-vx.x.x** folder execute the command **python vdt.py**.

```sh
root@vcenter-01v [ ~/vdt-v1.1.4 ]# python vdt.py 
_________________________
   RUNNING PULSE CHECK   

Today: Tuesday, December 27 20:42:42
Version: 1.1.4
Log Level: INFO

Provide password for administrator@vsphere.local: 
________________________
   VCENTER BASIC INFO   


BASIC:
	Current Time: 2022-12-27 20:42:47.424017
	vCenter Uptime: up 5:19
	vCenter Load Average: 0.04, 0.18, 0.22
	Number of CPUs: 2
	Total Memory: 11.7
	vCenter Hostname: vcenter-01v.pharmax.local
	vCenter PNID: vcenter-01v.pharmax.local
	vCenter IP Address: 192.168.5.2
	Proxy Configured: "no"
	NTP Servers: 192.168.5.1
	vCenter Node Type: vCenter with Embedded PSC
	vCenter Version: 8.0.0.10100 - 20920323
DETAILS:
	vCenter SSO Domain: vsphere.local
	vCenter AD Domain: No DOMAIN
	Number of ESXi Hosts: 3
	Number of Virtual Machines: 128
	Number of Clusters: 3
	Disabled Plugins: None

__________________
   VC DNS CHECK   

[FAIL]	Running script: /root/vdt-v1.1.4/scripts/_vc_dns.sh timed out.  Please re-run with --force.
__________________________
   Lookup Service Check   

[FAIL]	Running script: /root/vdt-v1.1.4/scripts/lsreport.py timed out.  Please re-run with --force.
_________________
   VC AD CHECK   


Domain Report:
	No domain(s) detected

Domain Exclusion List:

	 None

DC Exclusion List:

	 None

__________________________
   VC CERTIFICATE CHECK   

[PASS]	ESXi Certificate Management Mode: vmca

Checking MACHINE_SSL_CERT

	[PASS]	Supported Signature Algorithm
	[PASS]	Certificate trust check
	[PASS]	Certificate expiration check
	[PASS]	Certificate SAN check

Checking Other Certificate Stores

    MACHINE
	[PASS]	Supported Signature Algorithm
	[PASS]	Certificate trust check
	[PASS]	Certificate expiration check
	[INFO]	Certificate SAN check
	DETAILS: SAN contains hostname but not IP.

    VSPHERE-WEBCLIENT
	[PASS]	Supported Signature Algorithm
	[PASS]	Certificate trust check
	[PASS]	Certificate expiration check
	[INFO]	Certificate SAN check
	DETAILS: SAN contains hostname but not IP.

    VPXD
	[PASS]	Supported Signature Algorithm
	[PASS]	Certificate trust check
	[PASS]	Certificate expiration check
	[INFO]	Certificate SAN check
	DETAILS: SAN contains hostname but not IP.

    VPXD-EXTENSION
	[PASS]	Supported Signature Algorithm
	[PASS]	Certificate trust check
	[PASS]	Certificate expiration check
	[PASS]	Check extended key usage
	[INFO]	Certificate SAN check
	DETAILS: SAN contains hostname but not IP.
	Checking VC Extension Thumbprints
		[PASS]	com.vmware.vim.eam Thumbprint Check
		[PASS]	com.vmware.rbd Thumbprint Check
		[PASS]	com.vmware.imagebuilder Thumbprint Check

    DATA-ENCIPHERMENT
	[PASS]	Supported Signature Algorithm
	[PASS]	Certificate trust check
	[PASS]	Certificate expiration check
	[INFO]	Certificate SAN check
	DETAILS: SAN contains hostname but not IP.

    SMS
	[PASS]	Supported Signature Algorithm
	[PASS]	Certificate expiration check

    BACKUP_STORE
	NOTE: 	If you do not need your old certs, you can delete this store.
	Command:  /usr/lib/vmware-vmafd/bin/vecs-cli store delete --name BACKUP_STORE


    HVC
	[PASS]	Supported Signature Algorithm
	[PASS]	Certificate trust check
	[PASS]	Certificate expiration check
	[INFO]	Certificate SAN check
	DETAILS: SAN contains hostname but not IP.

    WCP
	[PASS]	Supported Signature Algorithm
	[PASS]	Certificate trust check
	[PASS]	Certificate expiration check

Checking TRUSTED_ROOTS certificates

  Alias: 33081c78da512bdb7716bb537e7b5c9cb33e15ba
	[PASS]	Supported Signature Algorithm
	[PASS]	Certificate is self-signed
	[PASS]	Certificate expiration check
	[PASS]	Certificate is a CA

  Alias: 22e9c5e5d583b37ca1da12c5e71433136a7ea420
	[PASS]	Supported Signature Algorithm
	[PASS]	Certificate is self-signed
	[PASS]	Certificate expiration check
	[PASS]	Certificate is a CA

  Alias: 62a73d6d9de388d14f8faad569b4ff9532158f87
	[PASS]	Supported Signature Algorithm
	[PASS]	Certificate is self-signed
	[PASS]	Certificate expiration check
	[PASS]	Certificate is a CA

  Alias: 88849bc0f586e78423f17765703ee262c33b78a9
	[PASS]	Supported Signature Algorithm
	[PASS]	Certificate is self-signed
	[PASS]	Certificate expiration check
	[PASS]	Certificate is a CA

  Alias: 0d46751c3b70a1aeeff774f501f29722a31c25d3
	[PASS]	Supported Signature Algorithm
	[PASS]	Certificate is self-signed
	[PASS]	Certificate expiration check
	[PASS]	Certificate is a CA

  Alias: 0f6d4d3b8c71290e76b6b6c0661275f6f37b9ce0
	[PASS]	Supported Signature Algorithm
	[PASS]	Certificate is self-signed
	[PASS]	Certificate expiration check
	[PASS]	Certificate is a CA

  Alias: 3f9f239ead6b20745042fbbd8b93b9045c5071f0
	[PASS]	Supported Signature Algorithm
	[PASS]	Certificate is self-signed
	[PASS]	Certificate expiration check
	[PASS]	Certificate is a CA

  Alias: 0c9d572dd410ecb72bec5587854f3914cf8be65c
	[PASS]	Supported Signature Algorithm
	[PASS]	Certificate is self-signed
	[PASS]	Certificate expiration check
	[PASS]	Certificate is a CA

Checking local LDAP cert

    VMDIR CERT
	[PASS]	Certificate expiration check

Checking STS Certs

	[PASS]	Certificate expiration check

_________________
   VMdir Check   

[INFO]	VMdir database size: 70.39MB

[INFO]	VMdir Status Check (No partners)

[PASS]	VMdir State Check

[PASS]	VMdir Arguments Check


_____________________
   CORE FILE CHECK   


INFO:   

These core files are older than 72 hours.  consider deleting them
at your discretion to reduce the size of log bundles.


  FILES:
    	/storage/core/core.python.116972 Size: 275.04MB Last Modified: 2022-12-17T16:45:21
    	/storage/core/core.vpxd-worker.65394 Size: 653.33MB Last Modified: 2022-11-01T09:57:50
    	/storage/core/core.vpxd-worker.4747 Size: 723.52MB Last Modified: 2022-11-01T09:57:22

[INFO]	Number of core files: 3

[PASS]	Number of hprof files: 0

______________________________
   vCenter PostgresDB Check   

Top 10 Largest Tables:

      tablename     |  size   
  ------------------+---------
   vpx_proc_log     | 45 MB
   vpx_event_arg_6  | 30 MB
   vpx_event_arg_9  | 30 MB
   vpx_event_arg_5  | 27 MB
   pk_vpx_proc_log  | 11 MB
   vpx_event_arg_7  | 5912 kB
   vpx_event_arg_26 | 5488 kB
   vpx_event_arg_14 | 4904 kB
   vpx_event_arg_10 | 4760 kB
   vpx_hist_stat3_6 | 4576 kB
  
Total Postgres Size:
	208M	/storage/db/vpostgres/
	679M	/storage/seat/vpostgres/
	858M	Interpreted by vPostgres

________________
   DISK CHECK   

[PASS]	DISK CAPACITY

[PASS]	INODE USAGE

RESULT: [PASS]
Please see KB: https://kb.vmware.com/s/article/1003564

__________________
   VC NTP CHECK   


[PASS] NTP service is running

NTP Server Check

[PASS] 192.168.5.1

NTP Status Check

+-----------------------------------LEGEND-----------------------------------+
| remote: NTP peer server                                                    |
| refid: server that this peer gets its time from                            |
| when: number of seconds passed since last response                         |
| poll: poll interval in seconds                                             |
| delay: round-trip delay to the peer in milliseconds                        |
| offset: time difference between the server and client in milliseconds      |
+-----------------------------------PREFIX-----------------------------------+
| * Synchronized to this peer                                                |
| # Almost synchronized to this peer                                         |
| + Peer selected for possible synchronization                               |
| – Peer is a candidate for selection                                        |
| ~ Peer is statically configured                                            |
+----------------------------------------------------------------------------+
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
*192.168.5.1     198.137.202.32   3 u  373 1024  377    0.232  -64.346  16.835

RESULT: [PASS]

________________________
   vCenter Port Check   

Checking ports: 443, 389, 2012, 2020
For port information, please see KB: https://kb.vmware.com/s/article/52963

	[PASS]	Port check for host vcenter-01v.pharmax.local

________________________
   Root Account Check   

[PASS]	Root password never expires

_______________________
   VC SERVICES CHECK   

Printing only services that are stopped and should be started.
KB: https://kb.vmware.com/s/article/2109887


RESULT: [PASS]

__________________
   Syslog Check   

Remote Syslog config: None configured

[PASS]	Local Syslog Functional Check

________________
   VCHA CHECK   

[INFO]	VCHA is not enabled.

Report written to /var/log/vmware/vdt/vdt-report-2022-12-27-204242
Please send feedback / feature requests to project_pulse@vmware.com
```

Review the results in the window. The meaning, results and instructions for each test are self-explanatory.

#### Hasta Luego Amigos!
