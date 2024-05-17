---
title: 'Seguridad: Auditoría de bases de datos de Active Directory con NtdsAudit'
author: 'Jonathan Colon Feliciano'
date: 2023-02-20T08:39:31-04:00
draft: false
tags:
  - 'Active Directory'
  - 'Seguridad'
  - 'Auditación'
---

En esta ocasión estaré hablando sobre un método para realizar auditación del contenido de la base de datos (NTDS.dit) de Active Directory. Esta herramienta llamada NtdsAudit fue creada por Dionach, aquí les dejo el enlace para que puedan ver las funcionalidades de la aplicación:

- <https://github.com/dionach/NtdsAudit>

Según se explica en la pagina de la herramienta NtdsAudit es una aplicación que ayuda a auditar las bases de datos de Active Directory. Proporciona algunas estadísticas útiles relacionadas con cuentas y contraseñas. También se puede utilizar para extraer hashes de contraseñas para su posterior crackeo.

Bueno ponemos aun lado la explicación y nos movemos al propósito de este articulo que es utilizar esta herramienta en un ambiente de `Active Directory` (AD). Para comenzar necesitamos extraer el contenido de la base de dados de AD, para lograr esta tarea necesitamos utilizar el comando `ntdsutil` desde un servidor controlador de dominio `Domain Controller`.

`Paso 1:` Abrir consola de comando en modo Administrador

![Text](/img/2023/auditing-ad-ntds-db/runasadmin-cmd.webp#center)

`Paso 2:` ejecutar el comando `ntdsutil`

```cmd
Microsoft Windows [Version 10.0.17763.4010]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Users\Administrator>ntdsutil
ntdsutil:
C:\Users\Administrator>
```

`Paso 3:` ejecutar el comando `activate instance ntds`

```cmd
Microsoft Windows [Version 10.0.17763.4010]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Users\Administrator>ntdsutil
ntdsutil: activate instance ntds
Active instance set to `ntds`.
C:\Users\Administrator>
```

`Paso 4:` ejecutar el comando `ifm`

```cmd
Microsoft Windows [Version 10.0.17763.4010]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Users\Administrator>

ntdsutil: ifm
```

`Paso 5:` ejecutar el comando `create full c:\ntds_export`

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

`Paso 6:` Salir del entorno de ntdsutils

```cmd
Microsoft Windows [Version 10.0.17763.4010]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Users\Administrator>
ifm: quit
ntdsutil: quit
```

Es importante verificar el contenido de la carpeta ntds_export para validar que se guardo correctamente la data durante el proceso de exportación de la base de datos.

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

Una vez tengamos los archivos de la base de dato `ntds.dit` podemos utilizar la herramienta para proveer estadísticas relacionado a cuentas de usuario y las contraseñas.

Ahora es necesario descargar el archivo ejecutable de la aplicación que podemos conseguir en el siguiente enlace:

- <https://github.com/dionach/NtdsAudit/releases/latest>

![Text](/img/2023/auditing-ad-ntds-db/ntdsaudit_download.webp#center)

En mi caso copie el archivo `NtdsAudit.exe` en la misma carpeta donde están los archivos de la base de datos ntds.dit (ntds_export):

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

A continuación les dejo los parámetros del comando `NtdsAudit.exe`. Este comando tiene varias opciones que te permiten escoger que función especifica deseas utilizar.

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

Por último, solo es necesario ejecutar el comando NtdsAudit.exe apuntando a la base de datos `ntds.dit` y al registro de `SYSTEM`. Ambos archivos se encuentran dentro de la carpeta `ntds_export`.

Ejemplo:

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

Como pueden observar este comando se puede utilizar para realizar `Cybersecurity researching` :)

Espero este articulo les haya sido de ayuda. `Hasta luego!!!`

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/F1F8DEV80)
