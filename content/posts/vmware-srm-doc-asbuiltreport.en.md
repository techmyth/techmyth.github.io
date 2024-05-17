---
title: 'HomeLab: VMware SRM Documentation with AsBuiltReport'
date: '2021-11-25T08:51:00-04:00'
tags:
    - VMware
---

Hello everyone!

These last week I have been working on improving my coding skills taking into account that now the trend is `Software Defined Everything`. This time I have been spending time on PowerShell creating several reports through the AsBuildReport project created by `Tim Carman` [@tpcarman](https://twitter.com/tpcarman) [Link Here]({{< ref "/posts/homelab-ontap-documentacion-asbuiltreport.en.md" >}} "HomeLab: Automated NetApp Ontap Documentation with AsBuiltReport").

The report I helped finish this time is related to documenting VMware Site Recovery Manager installations. The report was initially created by `Matt Allford` [@mattallford](https://twitter.com/mattallford) and I took on the task of trying to finish the work that had already been started.

According to VMware’s documentation portal:

> `Site Recovery Manager (SRM)` is the leading disaster recovery management solution designed to minimize downtime in the event of a disaster. It provides policy-based management and automated coordination, and enables testing of centralized recovery plans without disruption. It is designed for virtual machines and is scalable to manage all applications in a vSphere environment.
>
> [VMware Documentation](https://www.vmware.com/latam/products/site-recovery-manager.html)

The report is in an initial state and in constant development, but I decided to release it publicly to receive recommendations or rather to encourage other developers to contribute to improve its content. The development website of the report is in Github I leave the link so you can see the scope and objective of the project.

- <https://github.com/AsBuiltReport/AsBuiltReport.VMware.SRM>

![TEXT](/img/ASBuiltReport_SRM.webp#center)

Now, to get started we need to meet the following requirements:

- Multi-platform Windows, Linux or MAC
- PowerShell v5.1+ ó v7
- AsBuiltReport.Core Module >= 1.1.0
- VMware PowerCLI Module >= 12.3+

This report uses PowerShell version 5.+ or PSCore 7, to validate the version we can use the `$PSVersionTable` variable from the PowerShell console:

```text
PS /home/rebelinux> $PSVersionTable


Name                           Value
----                           -----
PSVersion                      7.2.0
PSEdition                      Core
GitCommitId                    7.2.0
OS                             Linux 5.15.4-arch1-1 #1 SMP PREEMPT Sun, 21 Nov 2021 21:34:33 +0000
Platform                       Unix
PSCompatibleVersions           {1.0, 2.0, 3.0, 4.0…}
PSRemotingProtocolVersion      2.3
SerializationVersion           1.1.0.1
WSManStackVersion              3.0

PS /home/rebelinux> 
```

To validate if we have the required modules we can use the `Get-Module` command as shown in the following example:

```text
PS /home/rebelinux> Get-Module -ListAvailable -Name @('VMware.PowerCLI','AsBuiltReport.Core')


    Directory: /home/rebelinux/.local/share/powershell/Modules

ModuleType Version    PreRelease Name                                PSEdition ExportedCommands
---------- -------    ---------- ----                                --------- ----------------
Script     1.1.0                 AsBuiltReport.Core                  Desk      {New-AsBuiltReport...}
Manifest   12.4.1.18…            VMware.PowerCLI                     Desk      

PS /home/rebelinux>
```

If the command does not produce any result it means that the modules are not installed. To install the dependency use the `Install-Module` command:

```text
PS /home/rebelinux>  Install-Module -Name @('VMware.PowerCLI','AsBuiltReport.Core')

Installing package 'VMware.PowerCLI'

Copying unzipped package to '..\2052046370\VMware.PowerCLI'

[oooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo]

PS /home/rebelinux> 
```

Once the prerequisites are met continue with the installation of the main module `AsBuiltReport.VMware.SRM`. Since this report has not yet been publicly released in `PowerShell Gallery` you need to manually perform the installation. The first step is to download the code from Github portal [here](https://github.com/rebelinux/AsBuiltReport.VMware.SRM).

![Text](/img/SRM_Download.webp#center)

Once the code is downloaded the file needs to be unzipped, for this you can use the command `Expand-Archive`.

```text
PS /home/rebelinux/Downloads> Expand-Archive -Path ./AsBuiltReport.VMware.SRM.zip -DestinationPath .

PS /home/rebelinux/Downloads> ls AsBuiltReport.VMware.SRM

AsBuiltReport.VMware.SRM.json  AsBuiltReport.VMware.SRM.psm1       CHANGELOG.md  README.md  Src
AsBuiltReport.VMware.SRM.psd1  AsBuiltReport.VMware.SRM.Style.ps1  LICENSE       Samples
PS /home/rebelinux/Downloads> 
```

Then it is necessary to copy the unzipped folder `AsBuiltReport.VMware.SRM` to a path set in `$env:PSModulePath`. The last step to follow if on `Windows OS` is to open a PowerShell window and unlock the downloaded files using the `Unblock-File` command.

```text
$path = (Get-Module -Name AsBuiltReport.VMware.SRM -ListAvailable).ModuleBase; Unblock-File -Path $path\*.psd1; Unblock-File -Path $path\Src\Public\*.ps1; Unblock-File -Path $path\Src\Private\*.ps1
```

An optional requirement is to generate configuration files that allow you to set the organization parameters that are used to generate the report. This process generates JSON files that are used as templates so that you do not have to fill in repetitive information when generating reports.

#### Archivos de configuración (AsBuiltReport JSON)

The powershell cmdlet New-AsBuiltConfig allows you to generate the template that is used as the basis of the report. This template sets the non-technical parameters of the report.

```text
PS /home/rebelinux>  New-AsBuiltConfig

---------------------------------------------
 <        As Built Report Information      >
---------------------------------------------
Enter the name of the Author for this As Built Report [jocolon]: Jonathan Colon
```

```text
---------------------------------------------
 <           Company Information           >
---------------------------------------------
Would you like to enter Company information for the As Built Report? (y/n): y
Enter the Full Company Name: Zen PR Solutions
Enter the Company Short Name: ZENPR
Enter the Company Contact: Jonathan Colon
Enter the Company Email Address: jcolonf@zenprsolutions.com
Enter the Company Phone: XXX-XXX-XXXX
Enter the Company Address: Puerto Rico
```

```text
---------------------------------------------
 <            Email Configuration          >
---------------------------------------------
Would you like to enter SMTP configuration? (y/n): n
```

```text
----------------------------------------------
 <       As Built Report Configuration      >
----------------------------------------------
Would you like to save the As Built Report configuration file? (y/n): y
Enter a name for the As Built Report configuration file [AsBuiltReport]: HomeLab SRM Report
Enter the path to save the As Built Report configuration file [/home/rebelinux/AsBuiltReport]:

Name                           Value
----                           -----
Email                          {Port, Credentials, Server, To...}
Company                        {FullName, Contact, Phone, Email...}
UserFolder                     {Path}
Report                         {Author}


PS /home/rebelinux>
```

Once the process is done, a JSON file will be created with the following content:

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
                       "Path":  "/home/rebelinux/AsBuiltReport"
                   },
    "Report":  {
                   "Author":  "Jonathan Colon"
               }
}
```

The `New-AsBuiltReportConfig` command allows you to set the technical parameters of the report such as the verbose level and type of information.

```text
PS /home/rebelinux/ New-AsBuiltReportConfig VMware.SRM -FolderPath /home/rebelinux/AsBuiltReport/
```

Once the process is completed, a JSON file will be created with the following content:

```json
{
    "Report": {
        "Name": "VMware SRM As Built Report",
        "Version": "1.0",
        "Status": "Released",
        "ShowCoverPageImage": true,
        "ShowTableOfContents": true,
        "ShowHeaderFooter": true,
        "ShowTableCaptions": true
    },
    "Options": {
        "ShowDefinitionInfo": false
    },
    "InfoLevel": {
        "_comment_": "0 = Disabled, 1 = Enabled, 2 = Adv Summary, 3 = Detailed",
        "Summary": 1,
        "Protected": 1,
        "Recovery": 1,
        "ProtectionGroup": 1,
        "RecoveryPlan": 1,
        "InventoryMapping": 1
    },
    "HealthCheck": {
        "InventoryMapping": {
            "Status": true
        },
        "Protected": {
            "Status": true
        },
        "Recovery": {
            "Status": true
        }

    }
}

```

This configuration file can be used to specify the level of detail of the report as well as which report sessions are to be enabled.

Then you can generate the report using the command `New-AsBuiltReport -Report VMware.SRM -Target vCenter_FQDN\or_IP`. It is important to emphasize that it is required to use the `vCenter` server address as `Target`.

```text
PS /home/rebelinux/> New-AsBuiltReport -Report VMware.SRM -AsBuiltConfigFilePath /home/rebelinux/script/AsBuiltReport.json -OutputFolderPath /home/rebelinux/script -Target 192.168.5.2 -Format HTML -Verbose -EnableHealthCheck -Credential $cred -ReportConfigFilePath /home/rebelinux/script/AsBuiltReport.VMware.SRM.json

VERBOSE: Loading As Built Report configuration from '/home/rebelinux/script/AsBuiltReport.json'.
VERBOSE: Loading module from path '/home/rebelinux/.local/share/powershell/Modules/AsBuiltReport.VMware.SRM/AsBuiltReport.VMware.SRM.psm1'.
VERBOSE: Loading AsBuiltReport.VMware.SRM report configuration file from path '/home/rebelinux/script/AsBuiltReport.VMware.SRM.json'.
VERBOSE: Setting report filename to 'VMware SRM As Built Report'.
VERBOSE: [ 12:49:41:970 ] [ Document ] - Document 'VMware SRM As Built Report' processing started.
VERBOSE: [ 12:49:42:016 ] [ Document ] - Executing report style script from path '/home/rebelinux/.local/share/powershell/Modules/AsBuiltReport.VMware.SRM\AsBuiltReport.VMware.SRM.Style.ps1'.
VERBOSE: [ 12:49:42:103 ] [ Document ] - Processing paragraph 'VMware SRM As Built Report'.
VERBOSE: [ 12:49:42:105 ] [ Document ] - Processing blank line.
VERBOSE: [ 12:49:42:110 ] [ Document ] - Processing paragraph 'Zen PR Solutions'.
VERBOSE: [ 12:49:42:111 ] [ Document ] - Processing blank line.
VERBOSE: [ 12:49:42:112 ] [ Document ] - Processing table 'Cover Page'.
VERBOSE: [ 12:49:42:114 ] [ Document ] - Processing page break.
VERBOSE: [ 12:49:42:115 ] [ Document ] - Processing table of contents 'Table of Contents'.
VERBOSE: [ 12:49:42:116 ] [ Document ] - Processing page break.
VERBOSE: [ 12:49:43:531 ] [ Document ] - Connected to protected site vCenter: 192.168.5.2
VERBOSE: [ 12:50:30:517 ] [ Document ] - Connected to vcenter-03v.zenpr.local
VERBOSE: [ 12:50:31:820 ] [ Document ] - Connected to protected site SRM: srmhq-01v.zenpr.local
VERBOSE: [ 12:50:31:821 ] [ Document ] - Processing section 'VMware Site Recovery Manager - SRMHQ-01V.' started.
VERBOSE: [ 12:50:31:823 ] [ Document ] - Processing paragraph 'VMware Site Recovery Manager is a bu[..]'.
VERBOSE: [ 12:50:31:833 ] [ Document ] - Processing blank line.
VERBOSE: [ 12:50:31:835 ] [ Document ] - Summary InfoLevel set at 1.
VERBOSE: [ 12:50:36:372 ] [ Document ] - Processing table 'Placeholder Datastore Mappings - SRM-HQ'.
VERBOSE: [ 12:50:36:375 ] [ Document ] - Processing section 'Placeholder Datastore Mappings' completed.
VERBOSE: [ 12:50:36:376 ] [ Document ] - Processing section 'Inventory Mapping Summary' completed.
VERBOSE: [ 12:50:36:377 ] [ Document ] - Protected Site InfoLevel set at 1.
VERBOSE: [ 12:50:36:377 ] [ Document ] - Collecting SRM Protected Site information.
VERBOSE: [ 12:50:36:387 ] [ Document ] - Processing section 'Protected Site' started.
VERBOSE: [ 12:50:36:395 ] [ Document ] - Processing paragraph 'In a typical Site Recovery Manager i[..]'.
VERBOSE: [ 12:50:36:396 ] [ Document ] - Processing blank line.
VERBOSE: [ 12:50:36:397 ] [ Document ] - Processing paragraph 'The following section provides a sum[..]'.
VERBOSE: [ 12:50:36:398 ] [ Document ] - Processing blank line.
VERBOSE: [ 12:50:36:398 ] [ Document ] - Discovered Protected Site SRM-HQ.
VERBOSE: [ 12:50:36:480 ] [ Document ] - Processing table 'Protected Site Information - SRM-HQ'.
VERBOSE: [ 12:50:36:483 ] [ Document ] - Processing section 'Protected Site' completed.
VERBOSE: [ 12:50:36:483 ] [ Document ] - Recovery Site InfoLevel set at 1.
VERBOSE: [ 12:50:36:484 ] [ Document ] - Collecting SRM Recovery Site information.
VERBOSE: [ 12:50:36:535 ] [ Document ] - Processing section 'Recovery Site' started.
VERBOSE: [ 12:50:36:536 ] [ Document ] - Processing paragraph 'In a typical Site Recovery Manager i[..]'.
VERBOSE: [ 12:50:36:537 ] [ Document ] - Processing blank line.
VERBOSE: [ 12:50:36:538 ] [ Document ] - Processing paragraph 'The following section provides a sum[..]'.
VERBOSE: [ 12:50:36:539 ] [ Document ] - Processing blank line.
VERBOSE: [ 12:50:36:539 ] [ Document ] - Discovered Recovery Site SRM-EDGE.
VERBOSE: [ 12:50:36:551 ] [ Document ] - Processing table 'Recovery Site Information - SRM-EDGE'.
VERBOSE: [ 12:50:36:554 ] [ Document ] - Processing section 'Recovery Site' completed.
VERBOSE: [ 12:50:36:555 ] [ Document ] - Protection Group Site InfoLevel set at 2.
VERBOSE: [ 12:50:36:555 ] [ Document ] - Collecting SRM Protection Group information.
VERBOSE: [ 12:50:36:561 ] [ Document ] - Processing section 'Protection Groups Summary' started.
VERBOSE: [ 12:50:36:563 ] [ Document ] - Processing paragraph 'In Site Recovery Manager, protection[..]'.
VERBOSE: [ 12:50:36:566 ] [ Document ] - Processing blank line.
VERBOSE: [ 12:50:36:567 ] [ Document ] - Processing paragraph 'The following section provides a sum[..]'.
VERBOSE: [ 12:50:36:570 ] [ Document ] - Processing blank line.
VERBOSE: [ 12:50:36:673 ] [ Document ] - Discovered Protection Group ZENPR-DB-PG.
VERBOSE: [ 12:50:36:735 ] [ Document ] - Discovered Protection Group ZENPR-MON-PG.
VERBOSE: [ 12:50:36:789 ] [ Document ] - Discovered Protection Group ZENPR-WEB-PG.
VERBOSE: [ 12:50:36:841 ] [ Document ] - Discovered Protection Group ZENPR-APP-PG.
VERBOSE: [ 12:50:36:855 ] [ Document ] - Processing table 'Protection Group Information - ZENPR-APP-PG'.
VERBOSE: [ 12:50:36:858 ] [ Document ] - Processing section 'Protection Groups' started.
VERBOSE: [ 12:50:36:873 ] [ Document ] - Processing paragraph 'The following section provides detai[..]'.
VERBOSE: [ 12:50:36:874 ] [ Document ] - Processing blank line.
VERBOSE: [ 12:50:36:892 ] [ Document ] - Processing section 'VMRS Type Protection Groups' started.
VERBOSE: [ 12:50:38:635 ] [ Document ] - Processing table 'VRMS Protection Group - ZENPR-APP-PG'.
WARNING: [ 12:50:38:639 ] [ Document ] - List table captions are only supported on tables with a single row. Removing caption from table 'VRMS Protection Group - ZENPR-APP-PG'.
VERBOSE: [ 12:50:38:639 ] [ Document ] - Processing section 'VMRS Type Protection Groups' completed.
VERBOSE: [ 12:50:38:640 ] [ Document ] - Processing section 'SAN Type Protection Groups' started.
VERBOSE: [ 12:50:39:145 ] [ Document ] - Processing table 'SAN Protection Group - ZENPR-APP-PG'.
VERBOSE: [ 12:50:39:147 ] [ Document ] - Processing section 'SAN Type Protection Groups' completed.
VERBOSE: [ 12:50:39:154 ] [ Document ] - Processing section 'Virtual Machine Protection Properties Summary' started.
VERBOSE: [ 12:50:39:167 ] [ Document ] - Processing paragraph 'The following section provides detai[..]'.
VERBOSE: [ 12:50:39:168 ] [ Document ] - Processing blank line.
VERBOSE: [ 12:50:39:234 ] [ Document ] - Processing section 'ZENPR-DB-PG Protection Properties (PlaceHolder)' started.
VERBOSE: [ 12:50:39:804 ] [ Document ] - Processing table 'VM Recovery PlaceHolder - ZENPR-DB-PG'.
VERBOSE: [ 12:50:39:811 ] [ Document ] - Processing section 'ZENPR-DB-PG Protection Properties (PlaceHolder)' completed.
VERBOSE: [ 12:50:39:855 ] [ Document ] - Processing section 'ZENPR-MON-PG Protection Properties (PlaceHolder)' started.
VERBOSE: [ 12:50:40:156 ] [ Document ] - Processing table 'VM Recovery PlaceHolder - ZENPR-MON-PG'.
VERBOSE: [ 12:50:40:158 ] [ Document ] - Processing section 'ZENPR-MON-PG Protection Properties (PlaceHolder)' completed.
VERBOSE: [ 12:50:40:204 ] [ Document ] - Processing section 'ZENPR-WEB-PG Protection Properties (PlaceHolder)' started.
VERBOSE: [ 12:50:40:554 ] [ Document ] - Processing table 'VM Recovery PlaceHolder - ZENPR-WEB-PG'.
VERBOSE: [ 12:50:40:556 ] [ Document ] - Processing section 'ZENPR-WEB-PG Protection Properties (PlaceHolder)' completed.
VERBOSE: [ 12:50:41:181 ] [ Document ] - Processing table 'Recovery Plan Config - ZENPR-MON-RG'.
VERBOSE: [ 12:50:41:198 ] [ Document ] - Processing section 'Virtual Machine Recovery Settings Summary' started.
VERBOSE: [ 12:50:41:245 ] [ Document ] - Processing paragraph 'The following section provides detai[..]'.
VERBOSE: [ 12:50:41:246 ] [ Document ] - Processing blank line.
VERBOSE: [ 12:50:41:257 ] [ Document ] - Processing section 'ZENPR-DB-RG VM Recovery Settings' started.
VERBOSE: [ 12:50:41:349 ] [ Document ] - Discovered VM Setting Linux-DB-01V.
VERBOSE: [ 12:50:41:571 ] [ Document ] - Discovered VM Setting Linux-DB-02V.
VERBOSE: [ 12:50:41:616 ] [ Document ] - Processing table 'Virtual Machine Recovery Settings - ZENPR-DB-RG'.
VERBOSE: [ 12:50:41:619 ] [ Document ] - Processing section 'ZENPR-DB-RG VM Recovery Settings' completed.
VERBOSE: [ 12:50:41:649 ] [ Document ] - Processing section 'ZENPR-APP-RG VM Recovery Settings' started.
VERBOSE: [ 12:50:41:696 ] [ Document ] - Discovered VM Setting Linux-APP-01V.
VERBOSE: [ 12:50:41:706 ] [ Document ] - Processing table 'Virtual Machine Recovery Settings - ZENPR-APP-RG'.
VERBOSE: [ 12:50:41:709 ] [ Document ] - Processing section 'ZENPR-APP-RG VM Recovery Settings' completed.
VERBOSE: [ 12:50:41:721 ] [ Document ] - Processing section 'ZENPR-WEB-RG VM Recovery Settings' started.
VERBOSE: [ 12:50:41:766 ] [ Document ] - Discovered VM Setting Linux-WEB-01V.
VERBOSE: [ 12:50:41:777 ] [ Document ] - Processing table 'Virtual Machine Recovery Settings - ZENPR-WEB-RG'.
VERBOSE: [ 12:50:41:779 ] [ Document ] - Processing section 'ZENPR-WEB-RG VM Recovery Settings' completed.
VERBOSE: [ 12:50:41:791 ] [ Document ] - Processing section 'ZENPR-MON-RG VM Recovery Settings' started.
VERBOSE: [ 12:50:41:840 ] [ Document ] - Discovered VM Setting Linux-Mon-01v.
VERBOSE: [ 12:50:41:853 ] [ Document ] - Processing table 'Virtual Machine Recovery Settings - ZENPR-MON-RG'.
VERBOSE: [ 12:50:41:856 ] [ Document ] - Processing section 'ZENPR-MON-RG VM Recovery Settings' completed.
VERBOSE: [ 12:50:41:858 ] [ Document ] - Processing section 'Virtual Machine Recovery Settings Summary' completed.
VERBOSE: [ 12:50:41:860 ] [ Document ] - Processing section 'Recovery Plans Summary' completed.
VERBOSE: [ 12:50:41:862 ] [ Document ] - Processing section 'VMware Site Recovery Manager - SRMHQ-01V.' completed.
VERBOSE: [ 12:50:41:893 ] [ Document ] - Document 'VMware SRM As Built Report' processing completed.
VERBOSE: [ 12:50:41:897 ] [ Document ] - Total processing time '59.93' seconds.
VMware SRM As Built Report 'VMware SRM As Built Report' has been saved to '/home/rebelinux/script'.
PS /home/rebelinux/>
```

Here is an example of the generated report.

{{< embed-pdf url="./img/VMware-SRM-As-Built-Report.pdf" >}}

Additionally, I include several examples of how to invoke the report.

```text
# Generate a VMware SRM As Built Report for vCenter Server 'vcenter.zenpr.local' using specified credentials. Export report to HTML & DOCX formats. Use default report style. Append timestamp to report filename. Save reports to 'C:\Users\Jon\Documents'

PS C:\> New-AsBuiltReport -Report VMware.SRM -Target 'vcenter.zenpr.local' -Username 'administrator@vsphere.local' -Password 'P@ssw0rd' -Format Html,Word -OutputFolderPath 'C:\Users\Jon\Documents' -Timestamp

# Generate a VMware SRM As Built Report for vCenter Server 'vcenter.zenpr.local' using specified credentials and report configuration file. Export report to Text, HTML & DOCX formats. Use default report style. Save reports to 'C:\Users\Jon\Documents'. Display verbose messages to the console.

PS C:\> New-AsBuiltReport -Report VMware.SRM -Target 'vcenter.zenpr.local' -Username 'administrator@vsphere.local' -Password 'P@ssw0rd' -Format Text,Html,Word -OutputFolderPath 'C:\Users\Jon\Documents' -ReportConfigFilePath 'C:\Users\Jon\AsBuiltReport\AsBuiltReport.VMware.SRM.json' -Verbose

# Generate a VMware SRM As Built Report for vCenter Server 'vcenter.zenpr.local' using stored credentials. Export report to HTML & Text formats. Use default report style. Highlight environment issues within the report. Save reports to 'C:\Users\Jon\Documents'.

PS C:\> $Creds = Get-Credential
PS C:\> New-AsBuiltReport -Report VMware.SRM -Target 'vcenter.zenpr.local' -Credential $Creds -Format Html,Text -OutputFolderPath 'C:\Users\Jon\Documents' -EnableHealthCheck

# Generate a VMware SRM As Built Report for vCenter Server 'vcenter.zenpr.local' using specified credentials. Export report to HTML & DOCX formats. Use default report style. Reports are saved to the user profile folder by default. Attach and send reports via e-mail.

PS C:\> New-AsBuiltReport -Report VMware.SRM -Target 'vcenter.zenpr.local' -Username 'administrator@vsphere.local' -Password 'P@ssw0rd' -Format Html,Word -OutputFolderPath 'C:\Users\Jon\Documents' -SendEmail
```

### Hasta Luego Amigos!

![Text](/img/documentation-meme.webp#center)
