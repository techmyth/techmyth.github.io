---
title: 'Veeam – Powershell get Backup Repository Information'
date: '2021-08-15T19:39:00-04:00'
tags:
    - VEEAM
---

Hello everyone,

These last days I have been performing several support services related to Veeam Backup & Replication where I have had the opportunity to use more Veeam Powershell modules. So in this opportunity I will be showing you in a basic way how to get information related to the repositories connected to the Backup server. As always it is necessary to establish a connection to the Backup server using the **“Connect-VBRServer”** command.

```text
PS C:\Users\jocolon> Connect-VBRServer -Server veeam-vbr.pharmax.local -Credential (Get-Credential)

cmdlet Get-Credential at command pipeline position 1
Supply values for the following parameters:
User: pharmax\administrator
Password for user pharmax\administrator: ********

PS C:\Users\jocolon> 
```

After connecting to the server you can use the **“Get-VBRBackupRepository”** command to identify the repositories connected to the Backup server.

```text
PS C:\Users\jocolon> Get-VBRBackupRepository                                           

Name                      Type         Host            FriendlyPath    Description
----                      ----         ----            ------------    -----------
VEEAM-DD                  DDBoost      VEEAM-VBR.ph... ddboost://VE... Created by PHARMAX\administrator
ONTAP Snapshot            SanSnapsh... VEEAM-VBR.ph... ONTAP Storage   Primary storage snapshot only
VEEAM-HPE-StoreOnce-VSA   HPStoreOn... VEEAM-VBR.ph... storeonce://... Created by PHARMAX\administrator
Linux - Hardened Repos... LinuxLocal   veeam-lnx-px... /backup_data... Created by PHARMAX\jocolon

PS C:\Users\jocolon>
```

As you can see the result of the cmdlet seems a bit incomplete as it does not show some important information such as the used space. In order to obtain this information I provide you with a basic example that allows you to obtain more relevant information.

```powershell
$OutObj = @()
if ((Get-VBRServerSession).Server) {
    try {
        [Array]$BackupRepos = Get-VBRBackupRepository | Where-Object {$_.Type -ne "SanSnapshotOnly"}
        [Array]$ScaleOuts = Get-VBRBackupRepository -ScaleOut
        if ($ScaleOuts) {
            foreach ($ScaleOut in $ScaleOuts) {
                $Extents = Get-VBRRepositoryExtent -Repository $ScaleOut
                foreach ($Extent in $Extents) {
                    $BackupRepos = $BackupRepos + $Extent.repository
                }
            }
        }
        foreach ($BackupRepo in $BackupRepos) {
            $PercentFree = 0
            if (@($($BackupRepo.GetContainer().CachedTotalSpace.InGigabytes),$($BackupRepo.GetContainer().CachedFreeSpace.InGigabytes)) -ne 0) {
                $UsedSpace = ($($BackupRepo.GetContainer().CachedTotalSpace.InGigabytes-$($BackupRepo.GetContainer().CachedFreeSpace.InGigabytes)))
                if ($UsedSpace -ne 0) {
                    $PercentFree = ($UsedSpace/$($BackupRepo.GetContainer().CachedTotalSpace.InGigabytes)).tostring("P")
                }
            }
            $inObj = [ordered] @{
                'Name' = $BackupRepo.Name
                'Total Space' = "$($BackupRepo.GetContainer().CachedTotalSpace.InGigabytes)Gb"
                'Free Space' = "$($BackupRepo.GetContainer().CachedFreeSpace.InGigabytes)Gb"
                'Percent Used' = $PercentFree
                'Status' = Switch ($BackupRepo.IsUnavailable) {
                    'False' {'Available'}
                    'True' {'Unavailable'}
                    default {$BackupRepo.IsUnavailable}
                }
            }
            $OutObj += [pscustomobject]$inobj
        }
        $OutObj | Format-Table -Wrap -AutoSize
    }
    catch {
        Write-Output $_.Exception.Message
    }
}
```

In the following example I show you the output of the resulting code to get the information about the repositories connected to the Veeam Backup server.

```text
Name                                  Total Space Free Space Percent Used Status     
----                                  ----------- ---------- ------------ ------
VEEAM-DD                              351Gb       292Gb      16.81%       Available
VEEAM-HPE-StoreOnce-VSA               0Gb         0Gb        0            Unavailable
Linux - Hardened Repository           199Gb       198Gb      0.50%        Available
E - Backup Repository                 499Gb       461Gb      7.62%        Available
F - Backup Repository                 99Gb        99Gb       0            Available
E - Backup Repository - VEEAM-VBR-02V 99Gb        5Gb        94.95%       Available
F - Backup Repository - VEEAM-VBR-02V 99Gb        98Gb       1.01%        Available
```

I hope you enjoyed this post and find it useful in your professional journey.

## Hasta Luego Amigos!
