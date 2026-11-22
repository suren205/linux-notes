# Week 9 – RAID, Storage Spaces & iSCSI (Month 3 starts)

## Lab
- **DC01** – Windows Server 2025 (Domain Controller), now also a **storage server**
- **CLIENT01** – Windows 11, now uses a **network disk** from DC01
- New this week: 2 extra 10 GB virtual disks on DC01 (`DC01-Mirror1.vhdx`, `DC01-Mirror2.vhdx`)

## 1. RAID in simple words
A disk **will** die one day. RAID uses several disks together to be faster or safer.

| RAID | Idea | Disks | Space you get | 1 disk dies? |
|---|---|---|---|---|
| **0** (stripe) | split data across disks → fast | 2+ | 100% | ❌ all data lost |
| **1** (mirror) | the same data on 2 disks | 2 | ~50% | ✅ safe |
| **5** (parity) | data + "math backup" spread across disks | 3+ | ~67% (3 disks) | ✅ safe |

**RAID is NOT a backup.** RAID copies every change to all disks at once, including deletes and ransomware.
- RAID = keep working when a disk dies
- Backup = go back in time

## 2. Storage Spaces mirror (RAID 1 on Windows Server)
On the laptop (Hyper-V host):
```powershell
New-VHD -Path D:\VMs\Disks\DC01-Mirror1.vhdx -SizeBytes 10GB -Dynamic
New-VHD -Path D:\VMs\Disks\DC01-Mirror2.vhdx -SizeBytes 10GB -Dynamic
Add-VMHardDiskDrive -VMName DC01 -Path D:\VMs\Disks\DC01-Mirror1.vhdx
Add-VMHardDiskDrive -VMName DC01 -Path D:\VMs\Disks\DC01-Mirror2.vhdx
```
On DC01:
```powershell
Get-PhysicalDisk -CanPool $true | Select-Object FriendlyName, Size, CanPool
```
Then in **Server Manager → File and Storage Services → Volumes → Storage Pools**, in this order:
1. **Pool** (Primordial → New Storage Pool, both 10 GB disks)
2. **Virtual disk** (Layout: **Mirror**, Fixed, maximum size)
3. **Volume** (drive **F:**, label `Mirror`)

Like a house: ground → walls → roof. The New Volume wizard first said *"No related virtual disks exist"* because I started with the roof.

Result: pool 19 GB, mirror disk **8 GB**. A mirror saves every file twice, so you get about half.

## 3. Disk-failure drill 😈
```powershell
# DC01: save a file on the mirror
"Secret mirror test - do not lose me!" | Out-File F:\test.txt

# Laptop: "kill" one disk while DC01 is running
Get-VMHardDiskDrive -VMName DC01 | Where-Object Path -like "*Mirror2*" | Remove-VMHardDiskDrive

# DC01: the file is still there
Get-Content F:\test.txt
Get-VirtualDisk | Select-Object FriendlyName, HealthStatus, OperationalStatus
#   MirrorDisk  Warning  {Degraded, Incomplete}

# Laptop: plug in a "new" disk
Add-VMHardDiskDrive -VMName DC01 -Path D:\VMs\Disks\DC01-Mirror2.vhdx

# DC01: rebuild the mirror
Repair-VirtualDisk -FriendlyName MirrorDisk
Get-VirtualDisk | Select-Object FriendlyName, HealthStatus, OperationalStatus
#   MirrorDisk  Healthy  OK
```
**Degraded** = still working, but no second copy. Fix it fast, because if the other disk dies too, everything is lost.

## 4. PowerShell report (advice from a senior engineer: "learn PowerShell")
```powershell
Get-ADUser -Filter * -SearchBase "OU=KumarCorp,DC=kumar,DC=local" -Properties Department |
    Select-Object Name, SamAccountName, Department, Enabled |
    Sort-Object Department |
    Export-Csv C:\Scripts\staff-report.csv -NoTypeInformation
```
The report showed 5 users with **no department**. Fixed them fast:
```powershell
Set-ADUser ram -Department IT
Set-ADUser sita -Department HR
```
Reports help you find messy data that is hard to see in the GUI.

## 5. NAS vs SAN and iSCSI

| | NAS 📁 | SAN 💽 |
|---|---|---|
| Client sees | a shared folder (`\\dc01\HR`) | a disk (drive `G:`) |
| Shares | files | raw disk blocks |

**iSCSI** builds a SAN over a normal network.
- **Target** 🎯 = gives the disk (DC01)
- **Initiator** 🙋 = uses the disk (CLIENT01)
- **IQN** = the target's unique name, e.g. `iqn.1991-05.com.microsoft:dc01-client01target-target`

On DC01 (target):
```powershell
Install-WindowsFeature FS-iSCSITarget-Server -IncludeManagementTools
New-Item -Path F:\iSCSI -ItemType Directory
New-IscsiVirtualDisk -Path F:\iSCSI\Client01-Disk.vhdx -SizeBytes 5GB
New-IscsiServerTarget -TargetName Client01Target -InitiatorIds "DNSName:client01.kumar.local"
Add-IscsiVirtualDiskTargetMapping -TargetName Client01Target -Path F:\iSCSI\Client01-Disk.vhdx
Get-IscsiServerTarget | Select-Object TargetName, Status
```
On CLIENT01 (initiator):
```powershell
Start-Service msiscsi
Set-Service msiscsi -StartupType Automatic
New-IscsiTargetPortal -TargetPortalAddress 192.168.50.10
Get-IscsiTarget | Connect-IscsiTarget -IsPersistent $true
Initialize-Disk -Number 1 -PartitionStyle GPT
New-Partition -DiskNumber 1 -UseMaximumSize -DriveLetter G | Format-Volume -FileSystem NTFS -NewFileSystemLabel NetworkDisk
"Hello from CLIENT01 over iSCSI!" | Out-File G:\hello.txt
```
CLIENT01 shows G: as a **Fixed** (local) disk, but the data really lives on DC01's mirror (F:\iSCSI). ✅

## Problems I solved

| Problem | Cause | Fix |
|---|---|---|
| `An empty pipe element is not allowed` | typed `\| Export-Csv` alone on a new line | put the whole command on one line |
| staff-report.csv only had `"Length","35"` | Export-Csv had no data and saved my typed text | ran the full one-line export again |
| New Volume wizard: "No related virtual disks exist" | started with the volume | Pool → Virtual disk → Volume, in order |
| CLIENT01: "Unable to allocate 2048 MB of RAM" | only 1.8 GB free on the laptop | `wsl --shutdown`, closed apps, `Set-VMMemory -VMName CLIENT01 -StartupBytes 1GB` |
| `-Initiatorlds` not found | small **l** instead of capital **I** | copy-paste long commands |
| Ran disk commands on **DC01** instead of CLIENT01 | did not check `hostname` | errors stopped me before I touched the backup disk; always run `hostname` before disk commands ⚠️ |

## Key ideas
- **RAID 1** survives one dead disk; you get about half the space.
- **Degraded** = working but unprotected. Replace the disk and `Repair-VirtualDisk`.
- **RAID is not a backup.**
- **NAS** = files, **SAN** = disks. **iSCSI** = SAN over Ethernet.
- `hostname` before every dangerous command, especially disk commands.

## In my own words
This week I started Month 3. I built a mirror from two disks with Storage Spaces, then pulled one disk out while the server was running. My file was still there, and after I put the disk back, `Repair-VirtualDisk` made it healthy again.
Then I turned DC01 into an iSCSI storage server and gave CLIENT01 a 5 GB network disk that looks like a local drive.
I also made my first PowerShell report and used it to find and fix missing data, which is the skill a senior engineer told me to focus on.
The scariest moment: I almost ran disk commands on the wrong server. Now I never skip `hostname`.
