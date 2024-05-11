---
title: 'HomeLab: Active Directory Documentation using AsBuiltReport'
date: '2021-10-14T07:22:42-04:00'
tags:
    - HomeLab
    - Active Directory
---

Hello to all!

Earlier I wrote about the NetApp.Ontap report that I helped develop through the AsBuildReport project created by `Tim Carman` [@tpcarman](https://twitter.com/tpcarman) (Here the [link](http://192.168.7.40/2021/09/27/homelab-ontap-docs-asbuiltreport/)). Well this time I will be talking about another report that I have been working on during these last months related to building documentation about the `Active Directory (AD)` infrastructure. For many years I have done several implementations and/or consultancies related to this service and I have always had the intention of creating a condensed report on how this service infrastructure was designed, which for statistical purposes runs in more than 95% of the world’s companies.

Although I have to admit that there are a number of experts who have created several similar reports, a unique feature of this report is that it uses [AsBuildReport](https://www.asbuiltreport.com/) as the basis for generating the programming necessary to set up the report structure. So I as a programmer can concentrate on the technology I want to document and not on adding code to generate the formatting in Word or HTML.

This report is in an initial state and in constant development, but I decided to release it publicly to receive recommendations or rather to encourage other developers to contribute to improve its content. The development website of the report is in Github I leave the link so you can see the scope and objective of the project.

- <https://github.com/AsBuiltReport/AsBuiltReport.Microsoft.AD>

![Text](/img/AsbuildReport_AD.webp)

This report can only be run on Windows 10+ or Windows Server 2012+. Additionally PowerShell version 5.1 or PowerShell 7+ is required and the following modules are also required in order to generate the report:

- [AsBuiltReport.Microsoft.AD](https://www.powershellgallery.com/packages/AsBuiltReport.Microsoft.AD/0.3.0)
- [ActiveDirectory](https://docs.microsoft.com/en-us/powershell/module/activedirectory/?view=windowsserver2022-ps)
- [PSPKI](https://www.powershellgallery.com/packages/PSPKI/3.7.2)
- [GroupPolicy](https://docs.microsoft.com/en-us/powershell/module/grouppolicy/?view=windowsserver2019-ps)

This report uses PowerShell version 5.+ or PSCore 7, to validate the version we can use the variable `$PSVersionTable` from the PowerShell console:

```text
PS C:\Users\jocolon> $PSVersionTable

Name                           Value
----                           -----
PSVersion                      5.1.19041.1237
PSEdition                      Desktop
PSCompatibleVersions           {1.0, 2.0, 3.0, 4.0...}
BuildVersion                   10.0.19041.1237
CLRVersion                     4.0.30319.42000
WSManStackVersion              3.0
PSRemotingProtocolVersion      2.3
SerializationVersion           1.1.0.1


PS C:\Users\jocolon>
```

To validate if the required modules are present we can use the `Get-Module` command as shown in the following example:

```text
PS C:\Users\jocolon> Get-Module -ListAvailable -Name @('ActiveDirectory','GroupPolicy','PSPKI')

    Directory: C:\Program Files\WindowsPowerShell\Modules

ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Script     3.7.2      PSPKI                               {Set-CertificateExtension, Get-CertificationAuthorityDbSch...

    Directory: C:\WINDOWS\system32\WindowsPowerShell\v1.0\Modules

ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Manifest   1.0.1.0    ActiveDirectory                     {Add-ADCentralAccessPolicyMember...
Manifest   1.0.0.0    GroupPolicy                         {Backup-GPO, Block-GPInheritance...

PS C:\Users\jocolon>
```

If the command does not produce any result it means that none of the modules are installed. To install these dependencies we use the `Install-Module` command:

```text
Installation of the PSPKI module:
PS C:\WINDOWS\system32> Install-Module -Name PSPKI                                                                                                                                                                                                                                                                                Installing package 'PSPKI'                                                                                                                                                                                                                                                        Copying unzipped package to 'C:\Users\jocolon\AppData\Local\Temp\2052046370\PSPKI'                                                                                                                                                                                             [oooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo]    
PS C:\WINDOWS\system32>                                                                                                                                                                                                                                                                                 
```

To install the `ActiveDirectory` and `GroupPolicy` modules use the command `Add-WindowsCapability` for Windows 10 or the command `Install-WindowsFeature` for Windows Server 2012+.

```text
Installation of ActiveDirectory and GroupPolicy module:
PS C:\WINDOWS\system32> Add-WindowsCapability -online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'                                                                                                                                                                                                                                                                                                                                                                                                                                                    
Path          :
Online        : True
RestartNeeded : False

PS C:\WINDOWS\system32> Add-WindowsCapability -online -Name 'Rsat.GroupPolicy.Management.Tools~~~~0.0.1.0'

Path          :
Online        : True
RestartNeeded : False

PS C:\WINDOWS\system32>

```

Once we have installed the prerequisites we can proceed with the installation of the main module `AsBuiltReport.Microsoft.AD`.

```text
PS C:\WINDOWS\system32> Install-Module -Name AsBuiltReport.Microsoft.AD

Untrusted repository
You are installing the modules from an untrusted repository. If you trust this repository, change its InstallationPolicy value by running the Set-PSRepository cmdlet. Are you sure you want to install the modules from 'PSGallery'?
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is `N`): Y

    Directory: C:\Program Files\WindowsPowerShell\Modules

PS C:\WINDOWS\system32>
```

To validate if the module was properly installed you can use the `Get-Module` command.

```text
PS C:\WINDOWS\system32> Get-Module -ListAvailable -Name AsBuiltReport.Microsoft.AD


    Directory: C:\Users\jocolon\Documents\WindowsPowerShell\Modules


ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Script     0.4.0      AsBuiltReport.Microsoft.AD          Invoke-AsBuiltReport.Microsoft.AD


PS C:\WINDOWS\system32>
```

Note: As you can see, the module `0.4.0` version was installed.

An optional requirement is to build a configuration file that allows you to set the organization’s parameters that are used to generate the report. This process will generate a JSON file that is used as a template so that you do not have to fill in repetitive information when generating reports.

#### Configuration files (AsBuiltReport JSON)

The powershell cmdlet New-AsBuiltConfig allows you to generate the template that we will use as the basis of the report. This template sets the non-technical parameters of the report.

```text
PS C:\WINDOWS\system32> New-AsBuiltConfig

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
Enter a name for the As Built Report configuration file [AsBuiltReport]: HomeLab ONTAP Report
Enter the path to save the As Built Report configuration file [C:\Users\jocolon\AsBuiltReport]:

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
    `Email`:  {
                  `Port`:  null,
                  `Credentials`:  null,
                  `Server`:  null,
                  `To`:  null,
                  `From`:  null,
                  `UseSSL`:  null,
                  `Body`:  null
              },
    `Company`:  {
                    `FullName`:  `Zen PR Solutions`,
                    `Contact`:  `Jonathan Colon`,
                    `Phone`:  `787-203-2790`,
                    `Email`:  `jcolonf@zenprsolutions.com`,
                    `ShortName`:  `ZENPR`,
                    `Address`:  `Puerto Rico`
                },
    `UserFolder`:  {
                       `Path`:  `C:\\Users\\jocolon\\AsBuiltReport`
                   },
    `Report`:  {
                   `Author`:  `Jonathan Colon`
               }
}
```

The New-AsBuiltReportConfig command allows you to set the technical parameters of the report such as the level and type of information collected `verbose level`.

```text
PS C:\WINDOWS\system32> New-AsBuiltReportConfig Microsoft.AD -FolderPath C:\Users\jocolon\AsBuiltReport\
```

Once the process is completed, a JSON file will be created with the following content:

```json
{
    `Report`: {
        `Name`: `Microsoft AD As Built Report`,
        `Version`: `1.0`,
        `Status`: `Released`,
        `ShowCoverPageImage`: true,
        `ShowTableOfContents`: true,
        `ShowHeaderFooter`: true,
        `ShowTableCaptions`: true
    },
    `Options`: {

    },
    `InfoLevel`: {
        `_comment_`: `0 = Disabled, 1 = Enabled, 2 = Adv Summary, 3 = Detailed`,
        `Forest`: 1,
        `Domain`: 1,
        `DHCP`: 1,
        `DNS`: 1,
        `CA`: 0,
        `Security`: 0
    },
    `HealthCheck`: {
        `Domain`: {
            `GMSA`: true,
            `GPO`: true
        },
        `DomainController`: {
            `Diagnostic`: true,
            `Services`: true
        },
        `Site`: {
            `Replication`: true
        },
        `DNS`: {
            `Aging`: true
        },
        `DHCP`: {
            `Summary`: true,
            `Credential`: true,
            `Statistics`: true,
            `BP`: true
        }
    }
}
```

This configuration file can be used to specify the level of detail of the report as well as which specific report sessions are to be enabled.

Then we can generate the report using the command `New-AsBuiltReport -Report Microsoft.AD -Target DC\_FQDN`. It is important to emphasize that it is required that the computer where the report is generated is added to the AD domain to be documented. It is also required to use the `fully qualified domain name `(FQDN) of the server with the `Domain Controller` role within the AD `Forest`.

```text
PS C:\Users\jocolon>  New-AsBuiltReport -Report Microsoft.AD -Target server-dc-01v.zenpr.local -Format HTML -AsBuiltConfigFilePath C:\Users\jocolon\AsBuiltReport\AsBuiltReport.json -OutputFolderPath C:\Users\jocolon\AsBuiltReport\ -ReportConfigFilePath '.\AsBuiltReport\AsBuiltReport.Microsoft.AD.json' -Credential $cred -EnableHealthCheck               

Please wait while the Microsoft AD As Built Report is being generated.
WARNING: [ 20:03:36:175 ] [ Document ] - List table captions are only supported on tables with a single row. Removing caption from table 'Group Managed Service Accounts Information - ZENPR.LOCAL'.
WARNING: [ 20:03:52:062 ] [ Document ] - List table captions are only supported on tables with a single row. Removing caption from table 'AD Domain Controller Hardware Information - ZENPR.LOCAL'.
WARNING: [ 20:04:12:144 ] [ Document ] - List table captions are only supported on tables with a single row. Removing caption from table 'Site Replication Information - ZENPR.LOCAL'.
WARNING: [ 20:04:12:412 ] [ Document ] - List table captions are only supported on tables with a single row. Removing caption from table 'Site Replication Failure Information - ZENPR.LOCAL'.
WARNING: [ 20:05:53:997 ] [ Document ] - Error: Collecting Active Directory Group Policy Objects with Startup/Shutdown Script for domain ACAD.ZENPR.LOCAL.
WARNING: [ 20:06:21:709 ] [ Document ] - Cannot bind argument to parameter 'Rows' because it is an empty collection.
WARNING: [ 20:06:21:853 ] [ Document ] - List table captions are only supported on tables with a single row. Removing caption from table 'IPv4 Scope Failover Cofiguration Information - SERVER-DC-01V'.
WARNING: [ 20:06:22:399 ] [ Document ] - Cannot bind argument to parameter 'Rows' because it is an empty collection.
WARNING: [ 20:06:22:440 ] [ Document ] - Cannot bind argument to parameter 'Rows' because it is an empty collection.
WARNING: [ 20:06:22:525 ] [ Document ] - Cannot bind argument to parameter 'Rows' because it is an empty collection.
WARNING: [ 20:06:22:555 ] [ Document ] - Cannot bind argument to parameter 'Rows' because it is an empty collection.
WARNING: [ 20:06:22:589 ] [ Document ] - Cannot bind argument to parameter 'Rows' because it is an empty collection.
WARNING: [ 20:06:22:897 ] [ Document ] - Error: Retreiving DHCP Server IPv4 Scope Failover Setting from ACADE-DC-01V.
Microsoft AD As Built Report 'Microsoft AD As Built Report' has been saved to 'C:\Users\jocolon\AsBuiltReport\'.
PS C:\Users\jocolon>
```

Here is an example of the resulting Active Directory documentation.

{{< embed-pdf url=`./img/Sample-Microsoft-AD-As-Built-Report.pdf` >}}

Additionally I include several examples of how to invoke the documentation report.

```text
# Generate a Microsoft Active Directory As Built Report for Domain Controller Server 'admin-dc-01v.contoso.local' using specified credentials. Export report to HTML & DOCX formats. Use default report style. Append timestamp to report filename. Save reports to 'C:\Users\Jon\Documents'

PS C:\> New-AsBuiltReport -Report Microsoft.AD -Target 'admin-dc-01v.contoso.local' -Username 'administrator@contoso.local' -Password 'P@ssw0rd' -Format Html,Word -OutputFolderPath 'C:\Users\Jon\Documents' -Timestamp

# Generate a Microsoft Active Directory As Built Report for Domain Controller Server 'admin-dc-01v.contoso.local' using specified credentials and report configuration file. Export report to Text, HTML & DOCX formats. Use default report style. Save reports to 'C:\Users\Jon\Documents'. Display verbose messages to the console.

PS C:\> New-AsBuiltReport -Report Microsoft.AD -Target 'admin-dc-01v.contoso.local' -Username 'administrator@contoso.local' -Password 'P@ssw0rd' -Format Text,Html,Word -OutputFolderPath 'C:\Users\Jon\Documents' -ReportConfigFilePath 'C:\Users\Jon\AsBuiltReport\AsBuiltReport.Microsoft.AD.json' -Verbose

# Generate a Microsoft Active Directory As Built Report for Domain Controller Server 'admin-dc-01v.contoso.local' using stored credentials. Export report to HTML & Text formats. Use default report style. Highlight environment issues within the report. Save reports to 'C:\Users\Jon\Documents'.

PS C:\> $Creds = Get-Credential
PS C:\> New-AsBuiltReport -Report Microsoft.AD -Target 'admin-dc-01v.contoso.local' -Credential $Creds -Format Html,Text -OutputFolderPath 'C:\Users\Jon\Documents' -EnableHealthCheck

# Generate a Microsoft Active Directory As Built Report for Domain Controller Server 'admin-dc-01v.contoso.local' using specified credentials. Export report to HTML & DOCX formats. Use default report style. Reports are saved to the user profile folder by default. Attach and send reports via e-mail.

PS C:\> New-AsBuiltReport -Report Microsoft.AD -Target 'admin-dc-01v.contoso.local' -Username 'administrator@contoso.local' -Password 'P@ssw0rd' -Format Html,Word -OutputFolderPath 'C:\Users\Jon\Documents' -SendEmail
```

### See you later folks!

![Text](/img/ad_meme.webp#center)
