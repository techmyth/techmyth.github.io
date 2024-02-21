---
title: 'HomeLab: Veeam VBR Documentation with AsBuiltReport'
date: '2024-02-20T10:50:00-04:00'
tags:
    - VEEAM
    - AsBuiltReport
    - Powershell
---

Hello everyone!

As you well know due to the impact of the Covid-19 pandemic I have not had much room to do anything other than stay confined to my house 😒. Even more so when so many of my family members have tested positive for the virus. So this situation has been the best excuse to get me into programming. Recently I have received many requests from various users and friends for me to build a report documenting the Backup application **Veeam Backup &amp; Replication**.

For this reason, I took on the task of developing yet another report, yes yet another one 🤣 and it is related to documenting **Veeam Backup &amp; Replication** implementations specifically for version **11+** onwards. This report uses as a base the AsBuildReport **framework** which is a project created by [Tim Carman](https://www.asbuiltreport.com/).

The source code of the report can be found in Github here I leave the link so you can see the scope of the project.

- <https://github.com/AsBuiltReport/AsBuiltReport.Veeam.VBR>

![Text](/img/Veeam_VBR_Portal.webp#center)

**Important: The report is currently available at *PowerShell Gallery**

Now, to get started we need to meet the following requirements:

- Windows platform only (Veeam powershell modules run on Windows only)
- PowerShell v5.1+
- AsBuiltReport.Core >= 1.3.0
- Veeam.Backup.PowerShell >= 1.0
- Veeam.Diagrammer >= 0.5.9

This report uses PowerShell version 5.+, to validate this use the **$PSVersionTable** variable from inside the PowerShell console:

```powershell
PS C:\Users\jocolon> $PSVersionTable

Name                           Value
----                           -----
PSVersion                      5.1.19041.3803
PSEdition                      Desktop
PSCompatibleVersions           {1.0, 2.0, 3.0, 4.0...}
BuildVersion                   10.0.19041.3803
CLRVersion                     4.0.30319.42000
WSManStackVersion              3.0
PSRemotingProtocolVersion      2.3
SerializationVersion           1.1.0.1

PS C:\Users\jocolon>
```

### Installation from PowerShellGallery

To validate whether the required Veeam modules are present, use the cmdlet **Get-Module -ListAvailable -Name @('Veeam.Backup.PowerShell') | Format-Table -AutoSize** as shown in the example below:

```powershell
PS C:\Users\jocolon> Get-Module -ListAvailable -Name @('Veeam.Backup.PowerShell') | Format-Table -AutoSize

    Directory: C:\Program Files\Veeam\Backup and Replication\Console

ModuleType Version     Name                    ExportedCommands
---------- -------     ----                    ----------------
Manifest   12.1.0.2131 Veeam.Backup.PowerShell {Get-VBRAmazonEC2VM, Get-VBRAzureVM, New-VBRAzureTag, New-VBRHealthCheckOptions...}

PS C:\Users\jocolon>
```

If the cmdlet does not produce any result it means that the modules are not installed. In order to install the **Veeam.Backup.PowerShell** modules, it is important to mention that they are available on the Veeam Backup server or on any device where the management console is installed.

**Reference:**

> The remote machine from which you run Veeam PowerShell commands must have the Veeam Backup & Replication Console installed. After you install the Veeam Backup & Replication Console, Veeam PowerShell module will be installed by default
>
> [Veeam PowerShell Reference](https://helpcenter.veeam.com/docs/backup/powershell/)

To install the **AsBuiltReport.Veeam.VBR** report from **PowerShell Gallery** use the **Install-Module -Name AsBuiltReport.Veeam.VBR** cmdlet:

```powershell
PS C:\Users\jocolon> Install-Module -Name AsBuiltReport.Veeam.VBR  
                                                                                                                        
Installing package 'AsBuiltReport.Veeam.VBR' 
[Downloaded 16.73 MB out of 23.76 MB.]

PS C:\Users\jocolon> 
```

To confirm whether all dependencies have been installed you can use the cmdlet: **Get-Module -ListAvailable -Name @('Veeam.Backup.PowerShell','PScribo', 'PScriboCharts', 'AsBuiltReport.Core', 'AsBuiltReport.Veeam.VBR', 'Veeam.Diagrammer') | select-object -Property Name, Version | Format-Table -AutoSize**.

```powershell
PS C:\Users\jocolon> Get-Module -ListAvailable -Name @('Veeam.Backup.PowerShell','PScribo', 'PScriboCharts', 'AsBuiltReport.Core', 'AsBuiltReport.Veeam.VBR', 'Veeam.Diagrammer') | select-object -Property Name, Version | Format-Table -AutoSize

Name                    Version    
----                    -------
AsBuiltReport.Veeam.VBR 0.8.5
Veeam.Diagrammer        0.5.9
AsBuiltReport.Core      1.3.0
PScribo                 0.10.0
PScriboCharts           0.9.0
Veeam.Backup.PowerShell 12.1.0.2131

PS C:\Users\jocolon>
```

If for some reason a module was not installed. You can install the required modules by using the following cmdlet: **Install-Module -SkipPublisherCheck -Force -Name @('PScribo', 'PScriboCharts', 'AsBuiltReport.Core', 'AsBuiltReport.Veeam.VBR', 'Veeam.Diagrammer')**

```powershell

PS C:\Users\jocolon> Install-Module -SkipPublisherCheck -Force -Name @('PScribo', 'PScriboCharts', 'AsBuiltReport.Core', 'AsBuiltReport.Veeam.VBR', 'Veeam.Diagrammer')

```

### Installation without Internet access

If for any reason, the computer where you are going to run the report does not have Internet access to download and install the required powershell modules from PowershellGallery. Since Powershell 5.1+ it is possible to save the previously installed modules on a computer so that they can be installed on the computer where the report will finally be run.

The **Save-Module** cmdlet allows you to save the required powershell modules to be installed on the machine that will finally run the report.

```powershell
PS C:\Users\jocolon> Save-Module -Path "C:\Users\Administrator\Downloads\OfflineModules" -Name @('PScribo', 'PScriboCharts', 'AsBuiltReport.Core', 'AsBuiltReport.Veeam.VBR', 'Veeam.Diagrammer')
```

![Text](/img/veeam_vbr_save_module.webp#center)

The files produced by the cmdlet **Save-Module** must be copied to the computer without access to the Internet in the folder **C:\Program Files\WindowsPowerShell\Modules**

![Text](/img/veeam_vbr_repor_modules_copy.webp#center)

To confirm whether all dependencies have been installed you can use the cmdlet: **Get-Module -ListAvailable -Name @('Veeam.Backup.PowerShell','PScribo', 'PScriboCharts', 'AsBuiltReport.Core', 'AsBuiltReport.Veeam.VBR', 'Veeam.Diagrammer') | select-object -Property Name, Version | Format-Table -AutoSize**.

```powershell
PS C:\Users\jocolon> Get-Module -ListAvailable -Name @('Veeam.Backup.PowerShell','PScribo', 'PScriboCharts', 'AsBuiltReport.Core', 'AsBuiltReport.Veeam.VBR', 'Veeam.Diagrammer') | select-object -Property Name, Version | Format-Table -AutoSize

Name                    Version    
----                    -------
AsBuiltReport.Veeam.VBR 0.8.5
Veeam.Diagrammer        0.5.9
AsBuiltReport.Core      1.3.0
PScribo                 0.10.0
PScriboCharts           0.9.0
Veeam.Backup.PowerShell 12.1.0.2131

PS C:\Users\jocolon>
```

### Configuration files (AsBuiltReport JSON)

An optional requirement is to build the configuration files that allow you to set the organization parameters that are used for report generation. This process generates JSON files that are used as templates **templates** so that you do not have to fill in repetitive information when generating the reports.

The powershell cmdlet **New-AsBuiltConfig** allows you to generate the template that you will use as the basis of the report. This template sets the non-technical parameters of the report.

```powershell
PS C:\Users\jocolon>  New-AsBuiltConfig

---------------------------------------------
 <        As Built Report Information      >
---------------------------------------------
Enter the name of the Author for this As Built Report [jocolon]: Jonathan Colon
```

```powershell
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

```powershell
---------------------------------------------
 <            Email Configuration          >
---------------------------------------------
Would you like to enter SMTP configuration? (y/n): n
```

```powershell
----------------------------------------------
 <       As Built Report Configuration      >
----------------------------------------------
Would you like to save the As Built Report configuration file? (y/n): y
Enter a name for the As Built Report configuration file [AsBuiltReport]: HomeLab Veeam Report
Enter the path to save the As Built Report configuration file [C:\Users\jocolon\AsBuiltReport]:

Name                           Value
----                           -----
Email                          {Port, Credentials, Server, To...}
Company                        {FullName, Contact, Phone, Email...}
UserFolder                     {Path}
Report                         {Author}


PS C:\Users\jocolon>
```

Once the process is completed, a JSON file will be created with the following content:

```json
{
  "Company": {
    "FullName": "As Built Report",
    "Phone": "888-888-8888",
    "Address": "Saun Juan, Pr",
    "ShortName": "ZENPR",
    "Contact": "Jonathan Colon",
    "Email": "jcolonf@zenprsolutions.com"
  },
  "Email": {
    "Credentials": true,
    "Body": "Reporte",
    "From": "jcolonf@zenprsolutions.com",
    "UseSSL": true,
    "Server": "smtp.gmail.com",
    "To": [
      "rebelinux@gmail.com"
    ],
    "Port": "587"
  },
  "Report": {
    "Author": "As Built Report"
  },
  "UserFolder": {
    "Path": ".\\AsBuiltReport"
  }
}

```

The **New-AsBuiltReportConfig** command allows you to set the technical parameters of the report such as the level and type of information **verbose level**.

```powershell
PS C:\Users\jocolon> New-AsBuiltReportConfig Veeam.VBR -FolderPath C:\Users\jocolon\AsBuiltReport\
```

Once the process is completed, a JSON file will be created with the following content:

```json
{
    "Report": {
        "Name": "Veeam Backup & Replication As Built Report",
        "Version": "1.0",
        "Status": "Released",
        "ShowCoverPageImage": true,
        "ShowTableOfContents": true,
        "ShowHeaderFooter": true,
        "ShowTableCaptions": true
    },
    "Options": {
        "BackupServerPort": 9392,
        "PSDefaultAuthentication": "Default",
        "EnableCharts": false,
        "EnableHardwareInventory": false,
        "EnableDiagrams": false
    },
    "InfoLevel": {
        "_comment_": "Please refer to the AsBuiltReport project contributing guide for information about how to define InfoLevels.",
        "_comment_": "0 = Disabled, 1 = Enabled, 2 = Adv Summary, 3 = Detailed",
        "Infrastructure": {
            "BackupServer": 1,
            "Proxy": 1,
            "Settings": 1,
            "BR": 1,
            "Licenses": 1,
            "SOBR": 1,
            "WANAccel": 1,
            "ServiceProvider": 1,
            "SureBackup": 1

        },
        "Tape": {
            "Server": 1,
            "Library": 1,
            "MediaPool": 1,
            "Vault": 1,
            "NDMP": 1
        },
        "Inventory": {
            "VI": 1,
            "PHY": 1,
            "FileShare": 1
        },
        "Storage": {
            "ONTAP": 1,
            "ISILON": 1
        },
        "Replication": {
            "FailoverPlan": 1,
            "Replica": 1
        },
        "CloudConnect": {
            "Certificate": 1,
            "PublicIP": 1,
            "CloudGateway": 1,
            "GatewayPools": 1,
            "Tenants": 1,
            "BackupStorage": 1,
            "ReplicaResources": 1
        },
        "Jobs": {
            "Backup": 1,
            "BackupCopy": 1,
            "Tape": 1,
            "Surebackup": 1,
            "Agent": 1,
            "FileShare": 1,
            "Replication": 1
        }
    },
    "HealthCheck": {
        "Infrastructure": {
            "BackupServer": true,
            "Proxy": true,
            "Settings": true,
            "BR": true,
            "SOBR": true,
            "Server": true,
            "Status": true,
            "BestPractice": true
        },
        "Tape": {
            "Status": true,
            "BestPractice": true
        },
        "Replication": {
            "Status": true,
            "BestPractice": true
        },
        "Security": {
            "BestPractice": true
        },
        "CloudConnect": {
            "Tenants": true,
            "BackupStorage": true,
            "ReplicaResources": true,
            "BestPractice": true
        },
        "Jobs": {
            "Status": true,
            "BestPractice": true
        }
    }
}
```

These configuration file can be used to specify the level of detail of the report as well as which report sections will be enabled.

### Report generation

The report can then be generated using the cmdlet: **New-AsBuiltReport -Report Veeam.VBR -Target Backup_Server_FQDN_or_IP -AsBuiltConfigFilePath AsBuiltReport.json -OutputFolderPath C:\Users\jocolon\AsBuiltReport\ -Credential $Cred -Format HTML -ReportConfigFilePath AsBuiltReport.Veeam.VBR.json -EnableHealthCheck -Verbose**. 

It is important to note that it is required to use the **IP Address or FQDN** of the **Veeam** backup server as **Target**.

```powershell
# Build the Credential Object
PS C:\Users\jocolon> $Creds = Get-Credential

# Run the report
PS C:\Users\jocolon> New-AsBuiltReport -Report Veeam.VBR -Target 'backup-server.fqdn.local' -AsBuiltConfigFilePath AsBuiltReport.json -OutputFolderPath C:\Users\jocolon\AsBuiltReport\ -Credential $Creds -Format HTML -ReportConfigFilePath AsBuiltReport.Veeam.VBR.json -EnableHealthCheck

Please wait while the Veeam VBR As Built Report is being generated.
WARNING: [ 09:52:51:541 ] [ Document ] - Please refer to the AsBuiltReport.Veeam.VBR github website for more detailed information about this project.
WARNING: [ 09:52:51:557 ] [ Document ] - Do not forget to update your report configuration file after each new version release.
WARNING: [ 09:52:51:557 ] [ Document ] - Documentation: https://github.com/AsBuiltReport/AsBuiltReport.Veeam.VBR
WARNING: [ 09:52:51:557 ] [ Document ] - Issues or bug reporting: https://github.com/AsBuiltReport/AsBuiltReport.Veeam.VBR/issues
WARNING: [ 09:52:51:682 ] [ Document ] - AsBuiltReport.Veeam.VBR 0.8.5 is currently installed.
Veeam VBR As Built Report 'Veeam Backup & Replication As Built Report' has been saved to 'C:\Users\jocolon\AsBuiltReport\'.

```

Once the cmdlet completes, a file will be generated in the format specified in **-Format** inside the folder specified in **-OutputFolderPath**.

Here is an example of the resulting report.

{{< embed-pdf url="./img/Veeam-VBR-As-Built-Report.pdf" >}}

&nbsp;

Lastly, I have included several examples of how to invoke the report.

```powershell
# Generate a Veeam VBR As Built Report for Backup Server 'backup-server.fqdn.local' using specified credentials. Export report to HTML & DOCX formats. Use default report style. Append timestamp to report filename. Save reports to 'C:\Users\Jon\Documents'

PS C:\> New-AsBuiltReport -Report Veeam.VBR -Target "backup-server.fqdn.local" -Username 'Domain\veeam_admin' -Password 'P@ssw0rd' -Format Html,Word -OutputFolderPath 'C:\Users\Jon\Documents' -Timestamp

# Generate a Veeam VBR As Built Report for Backup Server 'backup-server.fqdn.local' using specified credentials and report configuration file. Export report to Text, HTML & DOCX formats. Use default report style. Save reports to 'C:\Users\Jon\Documents'. Display verbose messages to the console.

PS C:\> New-AsBuiltReport -Report Veeam.VBR -Target "backup-server.fqdn.local" -Username 'Domain\veeam_admin' -Password 'P@ssw0rd' -Format Text,Html,Word -OutputFolderPath 'C:\Users\Jon\Documents' -ReportConfigFilePath 'C:\Users\Jon\AsBuiltReport\AsBuiltReport.Veeam.VBR.json' -Verbose

# Generate a Veeam VBR As Built Report for Backup Server 'backup-server.fqdn.local' using stored credentials. Export report to HTML & Text formats. Use default report style. Highlight environment issues within the report. Save reports to 'C:\Users\Jon\Documents'.

PS C:\> $Creds = Get-Credential
PS C:\> New-AsBuiltReport -Report Veeam.VBR -Target "backup-server.fqdn.local" -Credential $Creds -Format Html,Text -OutputFolderPath 'C:\Users\Jon\Documents' -EnableHealthCheck

# Generate a Veeam VBR As Built Report for Backup Server 'backup-server.fqdn.local' using stored credentials. Export report to HTML & DOCX formats. Use default report style. Reports are saved to the user profile folder by default. Attach and send reports via e-mail.

PS C:\> New-AsBuiltReport -Report Veeam.VBR -Target "backup-server.fqdn.local" -Username 'Domain\veeam_admin' -Password 'P@ssw0rd' -Format Html,Word -OutputFolderPath 'C:\Users\Jon\Documents' -SendEmail
```

### Hasta Luego Amigos

![Text](/img/backup_meme.webp#center)
