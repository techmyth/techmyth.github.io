---
title: 'Security: Auditing Active Directory Database with NtdsAudit'
author: 'Jonathan Colon Feliciano'
date: 2023-02-20T08:39:31-04:00
draft: false
tags:
  - 'Active Directory'
  - 'Security'
  - 'Auditing'
---

This time I am going to talk about a method to perform audits of the contents of the Active Directory database (NTDS.dit). The tool called `NtdsAudit` was created by `Dionach`, here I leave the link so you can see the functionality of the application:

- <https://github.com/dionach/NtdsAudit>

As explained on the tool's web page `NtdsAudit` is an application that helps to audit Active Directory databases. It provides some useful statistics related to accounts and passwords. It can also be used to extract password hashes for later cracking.

Well we put aside the explanation and move to the purpose of this article which is to use this tool in an `Active Directory` (AD) environment. To start we need to extract the contents of the AD database, to accomplish this task we need to use the `ntdsutil` command from a domain controller server `Domain Controller`.

##### Step 1: Open command console in Administrator mode

![Text](/img/2023/auditing-ad-ntds-db/runasadmin-cmd.webp#center)

##### Step 2: Execute the command `ntdsutil`.

```cmd
Microsoft Windows [Version 10.0.17763.4010]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Users\Administrator>ntdsutil
ntdsutil:
C:\Users\Administrator>
```

##### Step 3: Execute the command `activate instance ntds`

```cmd
Microsoft Windows [Version 10.0.17763.4010]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Users\Administrator>ntdsutil
ntdsutil: activate instance ntds
Active instance set to `ntds`.
C:\Users\Administrator>
```

##### Step 4: Execute the command ifm

```cmd
Microsoft Windows [Version 10.0.17763.4010]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Users\Administrator>

ntdsutil: ifm
```

##### Step 5: Execute the command `create full c:\ntds_export`

```cmd
Microsoft Windows [Version 10.0.17763.4010]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Users\Administrator>
ntdsutil:
ifm:  Create Full c:\ntds_export
Creating snapshot...
Snapshot set {bb0d84bd-e35f-4d4b-ac33-3222b0696e36} generated successfully.
Snapshot {1d9e428c-26c4-4c0e-ba29-4e3e9d5678cd} mounted as C:\$SNAP_202302202041_VOLUMEC$\
Snapshot {1d9e428c-26c4-4c0e-ba29-4e3e9d5678cd} is already mounted.
Initiating DEFRAGMENTATION mode...
     Source Database: C:\$SNAP_202302202041_VOLUMEC$\Windows\NTDS\ntds.dit
     Target Database: c:\ntds_export\Active Directory\ntds.dit

                  Defragmentation  Status (complete)

          0    10   20   30   40   50   60   70   80   90  100
          |----|----|----|----|----|----|----|----|----|----|
          ...................................................

Copying registry files...
Copying c:\ntds_export\registry\SYSTEM
Copying c:\ntds_export\registry\SECURITY
Snapshot {1d9e428c-26c4-4c0e-ba29-4e3e9d5678cd} unmounted.
IFM media created successfully in c:\ntds_export
```

##### Step 6: Leaving the ntdsutils environment

```cmd
Microsoft Windows [Version 10.0.17763.4010]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Users\Administrator>
ifm: quit
ntdsutil: quit
```

It is important to review the contents of the `ntds_export` folder to validate if the data was saved correctly during the database export process.

```cmd
c:\NetApp>cd c:\ntds_export

c:\ntds_export>dir
 Volume in drive C has no label.
 Volume Serial Number is D80A-563E

 Directory of c:\ntds_export

02/20/2023  08:41 PM    <DIR>          .
02/20/2023  08:41 PM    <DIR>          ..
02/20/2023  08:41 PM    <DIR>          Active Directory
02/20/2023  08:41 PM    <DIR>          registry
               0 File(s)              0 bytes
               4 Dir(s)   8,462,774,272 bytes free

c:\ntds_export>
```

Once we have the `ntds.dit` database files we can use the tool to provide statistics related to user accounts and passwords.

Now it is necessary to download the executable file of the application which can be obtained from the following link:

- <https://github.com/dionach/NtdsAudit/releases/latest>

![Text](/img/2023/auditing-ad-ntds-db/ntdsaudit_download.webp#center)

In my case I copied the `NtdsAudit.exe` file into the same folder where the `ntds.dit` database files were exported (ntds_export):

```cmd
c:\ntds_export>dir
 Volume in drive C has no label.
 Volume Serial Number is D80A-563E

 Directory of c:\ntds_export

02/20/2023  09:19 PM    <DIR>          .
02/20/2023  09:19 PM    <DIR>          ..
02/20/2023  08:41 PM    <DIR>          Active Directory
02/19/2023  11:32 AM           702,464 `NtdsAudit.exe`
02/20/2023  08:41 PM    <DIR>          registry
               1 File(s)        702,464 bytes
               4 Dir(s)   8,457,658,368 bytes free

c:\ntds_export>
```

Here are the parameters of the `NtdsAudit.exe` command. This command has several options that allow you to choose which specific function you want to use.

```cmd
Usage:  [arguments] [options]

Arguments:
  NTDS file  The path of the NTDS.dit database to be audited, required.

Options:
  -v | --version               Show version information
  -h | --help                  Show help information
  -s | --system <file>         The path of the associated SYSTEM hive, required when using the pwdump option.
  -p | --pwdump <file>         The path to output hashes in pwdump format.
  -u | --users-csv <file>      The path to output user details in CSV format.
  -c | --computers-csv <file>  The path to output computer details in CSV format.
  --history-hashes             Include history hashes in the pdwump output.
  --dump-reversible <file>     The path to output clear text passwords, if reversible encryption is enabled.
  --wordlist                   The path to a wordlist of weak passwords for basic hash cracking. Warning, using this option is slow, the use of a dedicated password cracker, such as 'john', is recommended instead.
  --ou-filter-file <file>      The path to file containing a line separated list of OUs to which to limit user and computer results.
  --base-date <yyyyMMdd>       Specifies a custom date to be used as the base date in statistics. The last modified date of the NTDS file is used by default.
  --debug                      Show debug output.

WARNING: Use of the --pwdump option will result in decryption of password hashes using the System Key.
Sensitive information will be stored in memory and on disk. Ensure the pwdump file is handled appropriately
```

Finally, it is only necessary to run the NtdsAudit.exe command pointing to the `ntds.dit` database and `SYSTEM` registry file. Both files are located inside the `ntds_export` folder.

Example:

```cmd
c:\ntds_export> ntdsaudit `Active Directory\ntds.dit` -s registry\SYSTEM -p pwdump.txt -u users.csv

The base date used for statistics is 2/21/2023 12:41:21 AM

Account stats for: pharmax.local
  Disabled users _____________________________________________________     4 of  2887 (0.1%)
  Expired users ______________________________________________________     1 of  2887 (0%)
  Active users unused in 1 year ______________________________________  2877 of  2882 (99.8%)
  Active users unused in 90 days _____________________________________  2879 of  2882 (99.9%)
  Active users which do not require a password _______________________     0 of  2882 (0%)
  Active users with non-expiring passwords ___________________________    17 of  2882 (0.6%)
  Active users with password unchanged in 1 year _____________________    19 of  2882 (0.7%)
  Active users with password unchanged in 90 days ____________________  2882 of  2882 (100%)
  Active users with Administrator rights _____________________________     6 of  2882 (0.2%)
  Active users with Domain Admin rights ______________________________     5 of  2882 (0.2%)
  Active users with Enterprise Admin rights __________________________     2 of  2882 (0.1%)

  Disabled computers _________________________________________________     4 of   213 (1.9%)
  Active computers unused in 1 year __________________________________   167 of   209 (79.9%)
  Active computers unused in 90 days _________________________________   188 of   209 (90%)

Account stats for: acad.pharmax.local
  Disabled users _____________________________________________________     2 of     3 (66.7%)
  Expired users ______________________________________________________     0 of     3 (0%)
  Active users unused in 1 year ______________________________________     1 of     1 (100%)
  Active users unused in 90 days _____________________________________     1 of     1 (100%)
  Active users which do not require a password _______________________     0 of     1 (0%)
  Active users with non-expiring passwords ___________________________     0 of     1 (0%)
  Active users with password unchanged in 1 year _____________________     1 of     1 (100%)
  Active users with password unchanged in 90 days ____________________     1 of     1 (100%)
  Active users with Administrator rights _____________________________     0 of     1 (0%)
  Active users with Domain Admin rights ______________________________     0 of     1 (0%)
  Active users with Enterprise Admin rights __________________________     0 of     1 (0%)

  Disabled computers _________________________________________________     0 of     4 (0%)
  Active computers unused in 1 year __________________________________     1 of     4 (25%)
  Active computers unused in 90 days _________________________________     2 of     4 (50%)

Account stats for: uia.local
  Disabled users _____________________________________________________     2 of  2493 (0.1%)
  Expired users ______________________________________________________     0 of  2493 (0%)
  Active users unused in 1 year ______________________________________  2490 of  2491 (100%)
  Active users unused in 90 days _____________________________________  2491 of  2491 (100%)
  Active users which do not require a password _______________________     0 of  2491 (0%)
  Active users with non-expiring passwords ___________________________     1 of  2491 (0%)
  Active users with password unchanged in 1 year _____________________     2 of  2491 (0.1%)
  Active users with password unchanged in 90 days ____________________  2491 of  2491 (100%)
  Active users with Administrator rights _____________________________     0 of  2491 (0%)
  Active users with Domain Admin rights ______________________________     0 of  2491 (0%)
  Active users with Enterprise Admin rights __________________________     0 of  2491 (0%)

  Disabled computers _________________________________________________     0 of   100 (0%)
  Active computers unused in 1 year __________________________________    99 of   100 (99%)
  Active computers unused in 90 days _________________________________    99 of   100 (99%)

WARNING:
The NTDS file has been retrieved from a global catalog (GC) server. Whilst GCs store information for other domains, they only store password hashes for their primary domain.
Password hashes have only been dumped for the `pharmax.local` domain.
If you require password hashes for other domains, please obtain the NTDS and SYSTEM files for each domain.

Password stats for: pharmax.local
  Active users using LM hashing ______________________________________     0 of  2882 (0%)
  Active users with duplicate passwords ______________________________    17 of  2882 (0.6%)
  Active users with password stored using reversible encryption ______     0 of  2882 (0%)


c:\ntds_export>
```

As you can see this command can be used to perform `Security researching` :)

I hope this article has been helpful to you. `Hasta luego!!!`

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/F1F8DEV80)
