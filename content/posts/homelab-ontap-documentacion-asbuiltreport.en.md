---
title: 'HomeLab: Automated NetApp Ontap Documentation with AsBuiltReport'
date: '2021-09-27T15:05:57-04:00'
tags:
    - NetApp
---

Hello everyone!

This time I come to talk about a project that I have been working on for a few months. If you remember some time ago I wrote about the AsBuildReport tool specifically the module to document VMware vSphere based infrastructures see the post [here](http://192.168.7.40/2021/06/06/homelab-automated-vmware-infrastructure-documentation/). Well today I come to show you a module that I help to develop with the purpose of documenting NetApp storage devices specifically on the ONTAP operating system.

This report is in an initial state and in active development, but I decided to release it publicly in order to receive recommendations or rather to encourage other developers to contribute to improve its content. The development website of the report is in Github I leave the link so you can see the scope and objective of the project:

<https://github.com/AsBuiltReport/AsBuiltReport.NetApp.ONTAP>

![Text](/img/AsBuildReport_NetApp_ONTAP.webp#center)

I should make it clear that this report is not designed to replace or compete in any way with the **NetAppDocs** tool. I noticed many requests on the NetApp forum from various users who have the need to generate an updated report of their storage infrastructure. It is for this reason that I took the initiative to create a report available free of charge for NetApp customers and/or users. One advantage of using the **AsBuiltReport** project, is that it allows you to create the report in multiple formats (Word, HTML or Text) and it is also possible to automate the sending of the report via email.

Now, to get started we need to meet the following requirements:

- Multi-platform Windows, Linux o MAC
- PowerShell v5.1+ ó v7
- “NetApp PowerShell Toolkit” >= 9.9.1.2106 powershell module
- AsBuiltReport.Core >= 1.1.0 powershell module

This report uses PowerShell version 5.+ or PSCore 7, to validate the version you can use the $PSVersionTable variable from the PowerShell shell:

```text
PS /home/rebelinux> >$PSVersionTable
>
Name                           Value
----                           -----
>PSVersion>                      >7.1.4>
PSEdition                      Core
GitCommitId                    7.1.4
OS                             Linux 5.14.2-arch1-2 #1 SMP PREEMPT Thu, 09 Sep 2021 09:42:35 +0000
Platform                       Unix
PSCompatibleVersions           {1.0, 2.0, 3.0, 4.0…}
PSRemotingProtocolVersion      2.3
SerializationVersion           1.1.0.1
WSManStackVersion              3.0

PS /home/rebelinux>
```

The report was created specifically for the version of “NetApp PowerShell Toolkit” >= 9.9.1.2106. To validate what version you have or if it has been installed you can use the Get-Module command as shown in the following example:

Note: Additionally I validate the version of “AsBuiltReport.Core” which is an additional dependency to be able to generate the report.

```text
PS /home/rebelinux> >Get-Module> -ListAvailable -Name >@('AsBuiltReport.Core','Netapp.Ontap')>                                    

    Directory: /home/rebelinux/.local/share/powershell/Modules

ModuleType Version    PreRelease Name                                PSEdition ExportedCommands
---------- -------    ---------- ----                                --------- ----------------
Script     >1.1.0>                 >AsBuiltReport.Core>                  Desk      {New-AsBuiltReport, New-AsBuiltConfig}
Manifest   >9.9.1.2106>            >NetApp.ONTAP >                       Desk      Add-NaHelpInfoUri

PS /home/rebelinux>
```

If no result is produced it means that none of the modules are installed. The Install-Module command is used to install these dependencies:

```text
PS /home/rebelinux> >Install-Module> -Name >@('AsBuiltReport.Core','Netapp.Ontap')>
Installing 2 modules.
Installing Module 'NetApp.ONTAP' (2 of 2).
[ooooooooooooooooooooooooooooooooooooooooooooooooooo]
Installing package 'NetApp.ONTAP'
Downloaded 1.47 MB out of 14.60 MB.
[oooooooooo                                         ]
PS /home/rebelinux>
```

Then proceed to install the ONTAP report using the following command:

```text
PS /home/rebelinux> >Install-Module> -Name >AsBuiltReport.NetApp.ONTAP>
Installing 1 modules.
Installing Module 'AsBuiltReport.NetApp.ONTAP' (1 of 1).
[ooooooooooooooooooooooooooooooooooooooooooooooooooo]
Installing package 'AsBuiltReport.NetApp.ONTAP'
Downloaded 0.2 MB out of 0.88 MB.
[oooooooooo                                         ]
PS /home/rebelinux>
```

To validate if the installation was successful we can use the Get-Module command.

```text
PS /home/rebelinux> >Get-Module> -ListAvailable >"AsBuiltReport.NetApp.ONTAP"> 


    Directory: C:\Program Files\WindowsPowerShell\Modules

ModuleType Version    PreRelease Name                                PSEdition ExportedCommands
---------- -------    ---------- ----                                --------- ----------------
Script     >0.4.0>                 >AsBuiltReport.NetApp.ONTAP>          Desk      Invoke-AsBuiltReport.NetApp.ONTAP

PS /home/rebelinux>
```

Note: As you can see, the module 0.4.0 version was installed.

An optional requirement is to generate a configuration file that allows you to set the organization parameters that are used to build the report. This process creates a JSON files that are used as templates so that you do not have to fill in repetitive information when building the reports.

The powershell cmdlet **New-AsBuiltConfig** allows you to build the template that will be used as the basis of the report. This template sets the non-technical parameters of the report.

```text
PS C:\WINDOWS\system32> >New-AsBuiltConfig
>
---------------------------------------------
 <        As Built Report Information      >
---------------------------------------------
Enter the name of the Author for this As Built Report [jocolon]: >Jonathan Colon>
```

```text
---------------------------------------------
 <           Company Information           >
---------------------------------------------
Would you like to enter Company information for the As Built Report? (y/n): >y>
Enter the Full Company Name: >Zen PR Solutions>
Enter the Company Short Name: >ZENPR>
Enter the Company Contact: >Jonathan Colon>
Enter the Company Email Address: >jcolonf@zenprsolutions.com>
Enter the Company Phone: >XXX-XXX-XXXX>
Enter the Company Address:> Puerto Rico>
```

```text
---------------------------------------------
 <            Email Configuration          >
---------------------------------------------
Would you like to enter SMTP configuration? (y/n): >n>
```

```text
----------------------------------------------
 <       As Built Report Configuration      >
----------------------------------------------
Would you like to save the As Built Report configuration file? (y/n): >y>
Enter a name for the As Built Report configuration file [AsBuiltReport]: >HomeLab ONTAP Report>
Enter the path to save the As Built Report configuration file [>C:\Users\jocolon\AsBuiltReport>]:

Name                           Value
----                           -----
Email                          {Port, Credentials, Server, To...}
Company                        {FullName, Contact, Phone, Email...}
UserFolder                     {Path}
Report                         {Author}


PS C:\WINDOWS\system32>
```

Once the process is completed, a JSON file will be created with the following content:

```json
{
    "Email":  {
                  "Port":  null,
                  "Credentials":  null,
                  "Server":  null,
                  "To":  null,
                  "From":  null,
                  "UseSSL":  null,
                  "Body":  null
              },
    "Company":  {
                    "FullName":  "Zen PR Solutions",
                    "Contact":  "Jonathan Colon",
                    "Phone":  "787-203-2790",
                    "Email":  "jcolonf@zenprsolutions.com",
                    "ShortName":  "ZENPR",
                    "Address":  "Puerto Rico"
                },
    "UserFolder":  {
                       "Path":  "C:\\Users\\jocolon\\AsBuiltReport"
                   },
    "Report":  {
                   "Author":  "Jonathan Colon"
               }
}
```

The **New-AsBuiltReportConfig** command allows you to set the technical parameters of the report such as the verbose level and type of information.

```text
PS C:\WINDOWS\system32> >New-AsBuiltReportConfig NetApp.ONTAP> -FolderPath C:\Users\jocolon\AsBuiltReport\
```

Once the process is completed, a JSON file will be created with the following content:

```json
{
    "Report": {
        "Name": "NetApp ONTAP As Built Report",
        "Version": "0.3.0",
        "Status": "Released",
        "ShowCoverPageImage": false,
        "ShowTableOfContents": true,
        "ShowHeaderFooter": true,
        "ShowTableCaptions": true
    },
    "Options": {

    },
    "InfoLevel": {
        "_comment_": "0 = Disabled, 1 = Enabled",
        "Cluster": 1,
        "Node": 1,
        "Storage": 1,
        "License": 1,
        "Network": 1,
        "Vserver": 1,
        "Replication": 1,
        "Efficiency": 1,
        "Security": 1,
        "System": 1
    },
    "HealthCheck": {
        "Cluster": {
            "Summary": true,
            "HA": true,
            "AutoSupport": true
        },
        "Node": {
            "HW": true,
            "ServiceProcessor": true
        },
        "Storage": {
            "Aggr": true,
            "DiskStatus": true,
            "ShelfStatus": true,
            "Efficiency": true,
            "FabricPool": true
        },
        "Network": {
            "Port": true,
            "Interface": true
        },
        "License": {
            "RiskSummary": true,
            "ServiceProcessor": true
        },
        "Vserver": {
            "Status": true,
            "Iscsi": true,
            "FCP": true,
            "NFS": true,
            "Quota": true,
            "CIFS": true,
            "Snapshot": true
        },
        "Replication": {
            "Status": true,
            "ClusterPeer": true,
            "VserverPeer": true,
            "Relationship": true,
            "History": true,
            "Mediator": true
        },
        "System": {
            "NTP": true,
            "DNS": true,
            "EMS": true,
            "Backup": true
        },
        "Security": {
            "Users": true,
            "KMS": true
    }
    }
}
```

Then we can build the report using the command **“New-AsBuiltReport -Report NetApp.ONTAP -Target IP/FQDN”**.

```text
PS /home/rebelinux> >New-AsBuiltReport> -Report >NetApp.ONTAP> -AsBuiltConfigFilePath /home/rebelinux/script/AsBuiltReport.json -OutputFolderPath /home/rebelinux/script >-Target> >192.168.7.60> -Format HTML  >-EnableHealthCheck> -Credential $cred -ReportConfigFilePath /home/rebelinux/script/AsBuiltReport.NetApp.ONTAP.json               

WARNING: Encountered Error No 81 on volume NTAPSOL_LUN_1. Reason: Volume is offline
WARNING: Encountered Error No 81 on volume NTAPSOL_LUN_1. Reason: Volume is offline
WARNING: Encountered Error No 81 on volume NTAPSOL_LUN_1. Reason: Volume is offline
WARNING: Encountered Error No 81 on volume NTAPSOL_LUN_1. Reason: Volume is offline
WARNING: Encountered Error No 81 on volume NTAPSOL_LUN_1. Reason: Volume is offline
WARNING: Encountered Error No 81 on volume NTAPSOL_LUN_1. Reason: Volume is offline
WARNING: Encountered Error No 81 on volume NTAPSOL_LUN_1. Reason: Volume is offline
WARNING: [ 13:17:34:209 ] [ Document ] - List table captions are only supported on tables with a single row. Removing caption from table 'Vserver Lun Information - PHARMAX-HQ'.
WARNING: Encountered Error No 81 on volume NTAPSOL_LUN_1. Reason: Volume is offline
WARNING: Encountered Error No 81 on volume NTAPSOL_LUN_1. Reason: Volume is offline
WARNING: Encountered Error No 81 on volume NTAPSOL_LUN_1. Reason: Volume is offline
WARNING: Encountered Error No 81 on volume NTAPSOL_LUN_1. Reason: Volume is offline
WARNING: Encountered Error No 81 on volume NTAPSOL_LUN_1. Reason: Volume is offline
WARNING: Encountered Error No 81 on volume NTAPSOL_LUN_1. Reason: Volume is offline
WARNING: Encountered Error No 81 on volume NTAPSOL_LUN_1. Reason: Volume is offline
WARNING: Encountered Error No 81 on volume NTAPSOL_LUN_1. Reason: Volume is offline
WARNING: [ 13:17:36:487 ] [ Document ] - List table captions are only supported on tables with a single row. Removing caption from table 'Vserver Igroup Information - PHARMAX-HQ'.
WARNING: Encountered Error No 81 on volume NTAPSOL_LUN_1. Reason: Volume is offline
WARNING: [ 13:17:37:516 ] [ Document ] - List table captions are only supported on tables with a single row. Removing caption from table 'Vserver NFS Service Options Summary - PHARMAX-HQ'.
WARNING: [ 13:17:49:537 ] [ Document ] - List table captions are only supported on tables with a single row. Removing caption from table 'Vserver CIFS Service Information - PHARMAX-HQ'.
WARNING: [ 13:17:50:264 ] [ Document ] - List table captions are only supported on tables with a single row. Removing caption from table 'Vserver CIFS Service Security Information - PHARMAX-HQ'.
WARNING: [ 13:17:51:834 ] [ Document ] - List table captions are only supported on tables with a single row. Removing caption from table 'Vserver CIFS Service Options Summary - PHARMAX-HQ'.
WARNING: [ 13:17:55:913 ] [ Document ] - List table captions are only supported on tables with a single row. Removing caption from table 'Replication - SnapMirror relationship Information - PHARMAX-HQ'.
WARNING: [ 13:17:56:662 ] [ Document ] - List table captions are only supported on tables with a single row. Removing caption from table 'Replication - SnapMirror Destination (List-Destinations) Information - PHARMAX-HQ'.
>NetApp ONTAP As Built Report 'NetApp ONTAP As Built Report' has been saved to '/home/rebelinux/script'.
>PS /home/rebelinux> 
```

Here is an example of the resulting report.

{{< embed-pdf url="./img/Sample-NetApp-ONTAP-As-Built-Report.pdf" >}}

Additionally I include several command line examples of how to generate the report. I hope you like it!

```text
# Generate a NetApp ONTAP As Built Report for Cluster array '192.168.7.60' using specified credentials. Export report to HTML & DOCX formats. Use default report style. Append timestamp to report filename. Save reports to 'C:\Users\Jon\Documents'>

PS C:\> New-AsBuiltReport -Report NetApp.ONTAP -Target 192.168.7.60 -Username 'admin' -Password 'P@ssw0rd' -Format Html,Word -OutputFolderPath 'C:\Users\Jon\Documents' -Timestamp>

# Generate a NetApp ONTAP As Built Report for Cluster array 192.168.7.60 using specified credentials and report configuration file. Export report to Text, HTML & DOCX formats. Use default report style. Save reports to 'C:\Users\Jon\Documents'. Display verbose messages to the console.>

PS C:\> New-AsBuiltReport -Report NetApp.ONTAP -Target 192.168.7.60 -Username 'admin' -Password 'P@ssw0rd' -Format Text,Html,Word -OutputFolderPath 'C:\Users\Jon\Documents' -ReportConfigFilePath 'C:\Users\Jon\AsBuiltReport\AsBuiltReport.NetApp.ONTAP.json' -Verbose>

# Generate a NetApp ONTAP As Built Report for Cluster array 192.168.7.60 using stored credentials. Export report to HTML & Text formats. Use default report style. Highlight environment issues within the report. Save reports to 'C:\Users\Jon\Documents'.>

PS C:\> $Creds = Get-Credential
PS C:\> New-AsBuiltReport -Report NetApp.ONTAP -Target 192.168.7.60 -Credential $Creds -Format Html,Text -OutputFolderPath 'C:\Users\Jon\Documents' -EnableHealthCheck

# Generate a NetApp ONTAP As Built Report for Cluster array 192.168.7.60 using stored credentials. Export report to HTML & DOCX formats. Use default report style. Reports are saved to the user profile folder by default. Attach and send reports via e-mail.>

PS C:\> New-AsBuiltReport -Report NetApp.ONTAP -Target 192.168.7.60 -Username 'admin' -Password 'P@ssw0rd' -Format Html,Word -OutputFolderPath 'C:\Users\Jon\Documents' -SendEmail>
```

### See you later Amigos

![Text](/img/infrastructure-so-complex-no-body-that-knows-how-it-was-35156082.webp#center)
