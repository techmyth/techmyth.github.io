---
title: 'HomeLab: Veeam VBR Documentation with AsBuiltReport'
date: '2021-12-22T10:50:00-04:00'
tags:
    - VEEAM
---

Hello everyone!

As you know, due to the impact of the Covid-19 pandemic, I haven’t had much room to do anything but stay confined to my home 😒. Another reason is that many of my family members have tested positive for the virus. So this situation has been the best excuse for me to get into development. Recently I have received many requests from users and friends to develop a report to document the Backup application made by “Veeam”.

For this reason, I took on the task of developing one more report, yes one more 🤣 and it is related to the documentation of “Veeam Backup & Replication” implementations. Currently only the latest version of Veeam is supported but I will evaluate in the future the possibility of adding support for versions prior to 11. This project is based on the AsBuildReport framework created by “**Tim Carman”** @tpcarman ([Project Page](https://www.asbuiltreport.com/)).

The report is in an early stage and under constant development, but I have decided to release it to the public to receive recommendations or rather to encourage other developers to contribute to improve its content. The code of the report is in Github so I leave here the link in order that you can see the status and objective of the project.

- <https://github.com/AsBuiltReport/AsBuiltReport.Veeam.VBR>

![Text](/img/Veeam_VBR_Portal.webp#center)

**Important: The module is now published in the PowerShell Gallery.**

Now, to get started, the following requirements must be met:

- Windows OS (Veeam modules run on Windows only)
- PowerShell v5.1+
- AsBuiltReport.Core module &gt;= 1.1.0
- Veeam.Backup.PowerShell module &gt;= 1.0
- SQLServer module &gt;+ 21.1

To validate the Powershell version, the variable **“$PSVersionTable”** can be used from a PowerShell console:

```text
PS C:\Users\jocolon> $PSVersionTable


Name                           Value
----                           -----
PSVersion                      5.1.19041.1320
PSEdition                      Desktop
PSCompatibleVersions           {1.0, 2.0, 3.0, 4.0...}
BuildVersion                   10.0.19041.1320
CLRVersion                     4.0.30319.42000
WSManStackVersion              3.0
PSRemotingProtocolVersion      2.3
SerializationVersion           1.1.0.1


PS C:\Users\jocolon>
```

To confirm if the required Veeam modules are present, the **“Get-Module”** cmdlet can be used as shown in the following example:

```text
PS C:\Users\jocolon> Get-Module -ListAvailable -Name @('Veeam.Backup.PowerShell')

    Directory: C:\Program Files\Veeam\Backup and Replication\Console


ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Manifest   1.0        Veeam.Backup.PowerShell             {Get-VBRComputerFileProxyServe....}


PS C:\Users\jocolon> 
```

If the command does not return any results, it means that the module is not installed. It is important to point out that the Veeam.Backup.PowerShell modules are available on the Veeam Backup server or on any device where the management console is already installed. Reference:

> The remote machine from which you run Veeam PowerShell commands must have the Veeam Backup &amp; Replication Console installed. After you install the Veeam Backup &amp; Replication Console, Veeam PowerShell module will be installed by default
>
> [Veeam PowerShell Reference](https://helpcenter.veeam.com/docs/backup/powershell/)

To install the **AsBuiltReport.Veeam.VBR**  from **PowerShell Gallery** use the traditional **“Install-Module”** command:

```text
PS C:\Users\jocolon> Install-Module -Name AsBuiltReport.Veeam.VBR  
                                                                                                                        
Installing package 'AsBuiltReport.Veeam.VBR' [Installing dependent package 'SqlServer']  
  Installing package 'SqlServer' [Downloaded 16.73 MB out of 23.76 MB.]

PS C:\Users\jocolon> 
```

To confirm if all the dependencies have been installed you can use the **“Get-Module”** cmdlet.

```text
PS C:\Users\jocolon> Get-Module -ListAvailable -Name @('Veeam.Backup.PowerShell','SQLServer','AsBuiltReport.Veeam.VBR','AsBuiltReport.Core')     
                                                                                                                        

    Directory: C:\Users\jocolon\Documents\WindowsPowerShell\Modules

ModuleType   Version    Name
----------   -------    ----
    Script   1.1.0      AsBuiltReport.Core
    Script   0.3.0      AsBuiltReport.Veeam.VBR
    Script   21.1.18256 SqlServer
    Manifest 1.0        Veeam.Backup.PowerShell  

PS C:\Users\jocolon> 
```

If for some reason you can not install the report through PowerShell Gallery I leave here the manual installation method.

#### **Manual module install** (**Github**)

Once the prerequisites are met the installation of the main module “AsBuiltReport.Veeam.VBR” can continue. Since this report has not yet been publicly released in “PowerShell Gallery” I will perform the installation manually. The first step is to download the code from the Github portal [here](https://github.com/AsBuiltReport/AsBuiltReport.Veeam.VBR/archive/refs/heads/dev.zip).

![Text](/img/SRM_Download.webp#center)

Files can be downloaded through the Powershell console:

```text
$Source = "https://github.com/AsBuiltReport/AsBuiltReport.Veeam.VBR/archive/refs/heads/dev.zip"
$Destination = '.\AsBuiltReport.Veeam.VBR.zip'

PS C:\Users\jocolon> Invoke-WebRequest -Uri $Source -OutFile $Destination

 Writing web request
    Writing request stream... (Number of bytes written: 51202)

PS C:\Users\jocolon> ls AsBuiltReport.Veeam.VBR.zip

    Directory: C:\Users\jocolon

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----             1/7/2022 1:58 PM         86404 AsBuiltReport.Veeam.VBR.zip

PS C:\Users\jocolon>
```

Once the source code is downloaded, you need to unzip the file, use the **“Expand-Archive”** cmdlet to achieve this goal.

```text
PS C:\Users\jocolon> Expand-Archive -Path ./AsBuiltReport.Veeam.VBR.zip -DestinationPath .

PS C:\Users\jocolon> Rename-Item -Path .\AsBuiltReport.Veeam.VBR-dev -NewName .\AsBuiltReport.Veeam.VBR

PS C:\Users\jocolon> ls AsBuiltReport.Veeam.VBR

    Directory: C:\Users\jocolon\AsBuiltReport.Veeam.VBR-dev

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----          1/7/2022   2:08 PM                .github
d-----          1/7/2022   2:08 PM                .vscode
d-----          1/7/2022   2:08 PM                Src
-a----          1/5/2022  10:52 AM           1191 AsBuiltreport.Veeam.VBR.json
-a----          1/5/2022  10:52 AM           3996 AsBuiltReport.Veeam.VBR.psd1
-a----          1/5/2022  10:52 AM            542 AsBuiltReport.Veeam.VBR.psm1
-a----          1/5/2022  10:52 AM          56642 AsBuiltReport.Veeam.VBR.Style.ps1
-a----          1/5/2022  10:52 AM           1871 CHANGELOG.md
-a----          1/5/2022  10:52 AM           1070 LICENSE
-a----          1/5/2022  10:52 AM          11796 README.md

PS C:\Users\jocolon> 
```

Next it is required to copy the unzipped folder **“AsBuiltReport.Veeam.VBR”** to a path set in **$env:PSModulePath**. To do this, you must first identify the folder where the content will be copied.

```text
PS C:\Users\jocolon> $env:PSModulePath.split(";")

C:\Users\jocolon\Documents\WindowsPowerShell\Modules
C:\Program Files\WindowsPowerShell\Modules
C:\WINDOWS\system32\WindowsPowerShell\v1.0\Modules
C:\Program Files\Veeam\Backup and Replication\Console\
C:\Program Files\Veeam\Backup and Replication\Explorers\Exchange\
C:\Program Files\Veeam\Backup and Replication\Explorers\SharePoint\
C:\Program Files\Veeam\Backup and Replication\Explorers\SQL\
C:\Program Files\Veeam\Backup and Replication\Explorers\ActiveDirectory\
C:\Program Files\Veeam\Backup and Replication\Explorers\Oracle\
C:\Program Files\Veeam\Backup and Replication\Explorers\Teams\
c:\Users\jocolon\.vscode\extensions\ms-vscode.powershell-2021.12.0\modules

PS C:\Users\jocolon> Copy-Item -Path .\AsBuiltReport.Veeam.VBR\ -Destination "C:\Users\jocolon\Documents\WindowsPowerShell\Modules\AsBuiltReport.Veeam.VBR" -PassThru -Recurse

PS C:\Users\jocolon>
```

The last step is to open a PowerShell window and unlock the downloaded files with the **“Unblock-File”** cmdlet.

```text
PS C:\Users\jocolon> $path = (Get-Module -Name AsBuiltReport.Veeam.VBR -ListAvailable).ModuleBase; Unblock-File -Path $path\*.*

PS C:\Users\jocolon>


PS C:\Users\jocolon> Get-Module -Name AsBuiltReport.Veeam.VBR -ListAvailable


   Directory: C:\Users\jocolon\Documents\WindowsPowerShell\Modules


ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Script     0.1.0      AsBuiltReport.Veeam.VBR             {Get-AbrVbrBackupServerCertificat....}


PS C:\Users\jocolon> Import-Module AsBuiltReport.Veeam.VBR -force 

```

An optional requirement is to build the configuration files that allow you to set the organization parameters needed to create the report. This process builds a JSON file used as template so that you do not have to fill in repetitive information when building the reports.

#### **Configuration Files (AsBuiltReport JSON)**

The powershell cmdlet **New-AsBuiltConfig** allows you to build the template that we will use as the basis of the report. This template sets the non-technical parameters of the report.

```text
PS C:\Users\jocolon>  New-AsBuiltConfig

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
Enter a name for the As Built Report configuration file [AsBuiltReport]: AsBuiltReport
Enter the path to save the As Built Report configuration file [C:\Users\jocolon\AsBuiltReport]:

Name                           Value
----                           -----
Email                          {Port, Credentials, Server, To...}
Company                        {FullName, Contact, Phone, Email...}
UserFolder                     {Path}
Report                         {Author}


PS C:\Users\jocolon>
```

Once the process finishes, a JSON file with the following content will be created:

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
                    "Phone":  "787-203-XXXX",
                    "Email":  "jcolonf@zenprsolutions.com",
                    "ShortName":  "ZENPR",
                    "Address":  "Puerto Rico"
                },
    "UserFolder":  {
                       "Path":  "C:\Users\jocolon\AsBuiltReport"
                   },
    "Report":  {
                   "Author":  "Jonathan Colon"
               }
}
```

The **New-AsBuiltReportConfig** command allows you to set the technical parameters of the report, such as the type of data collected and the “verbose” level.

```text
PS C:\Users\jocolon> New-AsBuiltReportConfig Veeam.VBR -FolderPath C:\Users\jocolon\AsBuiltReport\
```

Once the process is completed, a JSON file will be created with the following content:

```json
{
    "Report": {
        "Name": "Veeam VBR As Built Report",
        "Version": "1.0",
        "Status": "Released",
        "ShowCoverPageImage": true,
        "ShowTableOfContents": true,
        "ShowHeaderFooter": true,
        "ShowTableCaptions": true
    },
    "Options": {

    },
    "InfoLevel": {
    "_comment_": "Please refer to the AsBuiltReport project contributing guide for information about how to define InfoLevels.",
        "_comment_": "0 = Disabled, 1 = Enabled, 2 = Adv Summary, 3 = Detailed",
        "Infrastructure": {
            "Section": 1,
            "BackupServer": 1,
            "Proxy": 1,
            "Settings": 1,
            "BR": 1,
            "Licenses": 1,
            "SOBR": 1,
            "WANAccel": 1,
            "SureBackup": 1

        },
        "Tape": {
            "Section": 1,
            "Server": 1,
            "Library": 1
        }
    },
    "HealthCheck": {
        "Infrastructure": {
            "Proxy": true,
            "Settings": true,
            "BR": true,
            "SOBR": true,
            "Server": true,
            "Status": true

        },
        "Tape": {
            "Status": true
        }

    }
}
```

The resulting configuration file can be used to specify the level of detail of the report, as well as the report sessions to be enabled.

Finally, the **“New-AsBuiltReport”** cmdlet is used to build the report. It is important to note that it is required to use the IP address or the FQDN of the Veeam Backup Server as **“Target”**.

```text
# Store Credential in a Variable
PS C:\Users\jocolon> $Credential = Get-Credential
PowerShell credential request
Enter your credentials.
User: Veeam_Admin
Password for user Veeam_Admin: ***********

# Final Command 
PS C:\Users\jocolon> New-AsBuiltReport -Report Veeam.VBR -Target veeam-vbr.pharmax.local -AsBuiltConfigFilePath C:\Users\jocolon\AsBuiltReport\AsBuiltReport.json -OutputFolderPath C:\Users\jocolon\AsBuiltReport\ -Credential $Credential -Format HTML -ReportConfigFilePath C:\Users\jocolon\AsBuiltReport\AsBuiltReport.Veeam.VBR.json -EnableHealthCheck -Verbose

VERBOSE: Loading As Built Report configuration from 'C:\Users\jocolon\AsBuiltReport\AsBuiltReport.json'.
VERBOSE: Loading module from path 'C:\Users\jocolon\Documents\WindowsPowerShell\Modules\AsBuiltReport.Veeam.VBR\AsBuiltReport.Veeam.VBR.psm1'.
VERBOSE: Loading AsBuiltReport.Veeam.VBR report configuration file from path 'C:\Users\jocolon\AsBuiltReport\AsBuiltReport.Veeam.VBR.json'.
VERBOSE: Setting report filename to 'Veeam VBR As Built Report'.
VERBOSE: [ 15:04:09:872 ] [ Document ] - Document 'Veeam VBR As Built Report' processing started.
VERBOSE: [ 15:04:10:005 ] [ Document ] - Executing report style script from path 'C:\Users\jocolon\Documents\WindowsPowerShell\Modules\AsBuiltReport.Veeam.VBR\AsBuiltReport.Veeam.VBR.Style.ps1'.
VERBOSE: [ 15:04:10:036 ] [ Document ] - Setting global document options.
VERBOSE: [ 15:04:10:056 ] [ Document ] - Enabling section/heading numbering.
VERBOSE: [ 15:04:10:057 ] [ Document ] - Setting default font(s) to 'Arial'.
VERBOSE: [ 15:04:10:061 ] [ Document ] - Setting page top margin to '25.05'mm.
VERBOSE: [ 15:04:10:067 ] [ Document ] - Setting page right margin to '25.05'mm.
VERBOSE: [ 15:04:10:069 ] [ Document ] - Setting page bottom margin to '25.05'mm.
VERBOSE: [ 15:04:10:070 ] [ Document ] - Setting page left margin to '25.05'mm.
VERBOSE: [ 15:04:10:072 ] [ Document ] - Setting page size to 'A4'.
VERBOSE: [ 15:04:10:074 ] [ Document ] - Setting page orientation to 'Portrait'.
VERBOSE: [ 15:04:10:079 ] [ Document ] - Setting page height to '297'mm.
VERBOSE: [ 15:04:10:081 ] [ Document ] - Setting page width to '210'mm.
VERBOSE: [ 15:04:10:083 ] [ Document ] - Setting document style 'Title'.
VERBOSE: [ 15:04:10:085 ] [ Document ] - Setting document style 'Title2'.
VERBOSE: [ 15:04:10:105 ] [ Document ] - Setting document style 'Title3'.
VERBOSE: [ 15:04:10:110 ] [ Document ] - Setting document style 'Heading1'.
VERBOSE: [ 15:04:10:117 ] [ Document ] - Setting document style 'Heading2'.
VERBOSE: [ 15:04:10:121 ] [ Document ] - Setting document style 'Heading3'.
VERBOSE: [ 15:04:10:126 ] [ Document ] - Setting document style 'Heading4'.
VERBOSE: [ 15:04:10:129 ] [ Document ] - Setting document style 'Heading5'.
VERBOSE: [ 15:04:10:134 ] [ Document ] - Setting document style 'Heading6'.
VERBOSE: [ 15:04:10:136 ] [ Document ] - Setting document style 'Normal'.
VERBOSE: [ 15:04:10:159 ] [ Document ] - Setting document style 'Caption'.
VERBOSE: [ 15:04:10:162 ] [ Document ] - Setting document style 'Header'.
VERBOSE: [ 15:04:10:174 ] [ Document ] - Setting document style 'Footer'.
VERBOSE: [ 15:04:10:176 ] [ Document ] - Setting document style 'TOC'.
VERBOSE: [ 15:04:10:179 ] [ Document ] - Setting document style 'TableDefaultHeading'.
VERBOSE: [ 15:04:10:182 ] [ Document ] - Setting document style 'TableDefaultRow'.
VERBOSE: [ 15:04:10:185 ] [ Document ] - Setting document style 'Critical'.
VERBOSE: [ 15:04:10:192 ] [ Document ] - Setting document style 'Warning'.
VERBOSE: [ 15:04:10:216 ] [ Document ] - Setting document style 'Info'.
VERBOSE: [ 15:04:10:238 ] [ Document ] - Setting document style 'OK'.
VERBOSE: [ 15:04:10:264 ] [ Document ] - Setting table style 'TableDefault'.
VERBOSE: [ 15:04:10:281 ] [ Document ] - Setting table style 'Borderless'.
VERBOSE: [ 15:04:10:285 ] [ Document ] - Processing document header started.
VERBOSE: [ 15:04:10:288 ] [ Document ] - Processing paragraph 'Veeam VBR As Built Report - v1.0'.
VERBOSE: [ 15:04:10:295 ] [ Document ] - Processing document header completed.
VERBOSE: [ 15:04:10:298 ] [ Document ] - Processing document footer started.
VERBOSE: [ 15:04:10:302 ] [ Document ] - Processing paragraph 'Page <!# PageNumber #!>'.
VERBOSE: [ 15:04:10:318 ] [ Document ] - Processing document footer completed.
VERBOSE: [ 15:04:10:320 ] [ Document ] - Processing blank line.
VERBOSE: [ 15:04:10:343 ] [ Document ] - Processing image 'Veeam Logo'.
VERBOSE: [ 15:04:10:394 ] [ Document ] - Processing blank line.
VERBOSE: [ 15:04:10:396 ] [ Document ] - Processing paragraph 'Veeam VBR As Built Report'.
VERBOSE: [ 15:04:10:400 ] [ Document ] - Processing blank line.
VERBOSE: [ 15:04:10:419 ] [ Document ] - Processing paragraph 'Zen Pr Solutions'.
VERBOSE: [ 15:04:10:421 ] [ Document ] - Processing blank line.
VERBOSE: [ 15:04:10:423 ] [ Document ] - Processing table 'Cover Page'.
VERBOSE: [ 15:04:10:449 ] [ Document ] - Processing page break.
VERBOSE: [ 15:04:10:453 ] [ Document ] - Processing table of contents 'Table of Contents'.
VERBOSE: [ 15:04:10:465 ] [ Document ] - Processing page break.
VERBOSE: [ 15:04:10:742 ] [ Document ] - Trying to import Veeam B&R modules.
VERBOSE: [ 15:04:10:993 ] [ Document ] - Identifying Veeam Powershell module version.
VERBOSE: [ 15:04:10:995 ] [ Document ] - Using Veeam Powershell module version 11.
VERBOSE: [ 15:04:11:231 ] [ Document ] - Establishing the initial connection to the Backup Server: veeam-vbr.pharmax.local.
VERBOSE: [ 15:04:11:233 ] [ Document ] - Looking for veeam existing server connection.
VERBOSE: [ 15:04:11:335 ] [ Document ] - Existing veeam server connection found
VERBOSE: [ 15:04:11:337 ] [ Document ] - Validating connection to veeam-vbr.pharmax.local
VERBOSE: [ 15:04:11:460 ] [ Document ] - Successfully connected to veeam-vbr.pharmax.local VBR Server.
VERBOSE: [ 15:04:11:462 ] [ Document ] - Processing section 'Implementation Report' started.
VERBOSE: [ 15:04:11:464 ] [ Document ] - Processing paragraph 'The following section provides a sum[..]'.
VERBOSE: [ 15:04:11:471 ] [ Document ] - Processing blank line.
VERBOSE: [ 15:04:11:486 ] [ Document ] - Backup Infrastructure InfoLevel set at 2.
VERBOSE: [ 15:04:11:488 ] [ Document ] - Processing section 'Backup Infrastructure Summary' started.
VERBOSE: [ 15:04:11:508 ] [ Document ] - Discovering Veeam VBR Infrastructure Summary from veeam-vbr.pharmax.local.
Output Truncated!
VERBOSE: [ 15:06:02:777 ] [ Document ] - Discovered Pharmax - Veeam Tape Vault Type Vault.
VERBOSE: [ 15:06:02:979 ] [ Document ] - Processing table 'Tape Vault - VEEAM-VBR'.
VERBOSE: [ 15:06:02:986 ] [ Document ] - Processing section 'Tape Vault Summary' completed.
VERBOSE: [ 15:06:02:989 ] [ Document ] - Processing section 'Tape Infrastructure Summary' completed.
VERBOSE: [ 15:06:02:991 ] [ Document ] - Processing section 'Implementation Report' completed.
VERBOSE: [ 15:06:03:130 ] [ Document ] - Document 'Veeam VBR As Built Report' processing completed.
VERBOSE: [ 15:06:03:141 ] [ Document ] - Total processing time '1.89' minutes.
Veeam VBR As Built Report 'Veeam VBR As Built Report' has been saved to 'C:\Users\jocolon\AsBuiltReport\'.
PS C:\Users\jocolon> 
```

Here is an example of the resulting report.

{{< embed-pdf url="./img/Veeam-VBR-As-Built-Report.pdf" >}}

I also include several options on how to build the report.

```text
# Generate a Veeam VBR As Built Report for Backup Server '192.168.7.60' using specified credentials. Export report to HTML & DOCX formats. Use default report style. Append timestamp to report filename. Save reports to 'C:\Users\Jon\Documents'

PS C:\> New-AsBuiltReport -Report Veeam.VBR -Target 192.168.7.60 -Username 'Domain\veeam_admin' -Password 'P@ssw0rd' -Format Html,Word -OutputFolderPath 'C:\Users\Jon\Documents' -Timestamp

# Generate a Veeam VBR As Built Report for Backup Server 192.168.7.60 using specified credentials and report configuration file. Export report to Text, HTML & DOCX formats. Use default report style. Save reports to 'C:\Users\Jon\Documents'. Display verbose messages to the console.

PS C:\> New-AsBuiltReport -Report Veeam.VBR -Target 192.168.7.60 -Username 'Domain\veeam_admin' -Password 'P@ssw0rd' -Format Text,Html,Word -OutputFolderPath 'C:\Users\Jon\Documents' -ReportConfigFilePath 'C:\Users\Jon\AsBuiltReport\AsBuiltReport.Veeam.VBR.json' -Verbose

# Generate a Veeam VBR As Built Report for Backup Server 192.168.7.60 using stored credentials. Export report to HTML & Text formats. Use default report style. Highlight environment issues within the report. Save reports to 'C:\Users\Jon\Documents'.

PS C:\> $Creds = Get-Credential
PS C:\> New-AsBuiltReport -Report Veeam.VBR -Target 192.168.7.60 -Credential $Creds -Format Html,Text -OutputFolderPath 'C:\Users\Jon\Documents' -EnableHealthCheck

# Generate a Veeam VBR As Built Report for Backup Server 192.168.7.60 using stored credentials. Export report to HTML & DOCX formats. Use default report style. Reports are saved to the user profile folder by default. Attach and send reports via e-mail.

PS C:\> New-AsBuiltReport -Report Veeam.VBR -Target 192.168.7.60 -Username 'Domain\veeam_admin' -Password 'P@ssw0rd' -Format Html,Word -OutputFolderPath 'C:\Users\Jon\Documents' -SendEmail
```

## Hasta Luego Amigos!

![Text](/img/backup_meme.webp#center)
