# Week 8 – Automation, Recovery & the Month 2 Project

## Lab
- **DC01** – Windows Server 2025, Domain Controller for `kumar.local` (+ DNS, DHCP)
- **CLIENT01** – Windows 11 Enterprise, joined to the domain
- New this week: a 2nd virtual disk on DC01 (**E: Backup**) and scripts in `C:\Scripts`

## 1. Bulk users from a CSV file
Creating 50 users in the GUI takes hours. With a CSV file and PowerShell it takes seconds.

`C:\Scripts\newusers.csv`
```
First,Last,Username,Dept
Anil,Shrestha,anil,Sales
Maya,Thapa,maya,HR
Bikash,Rai,bikash,IT
Puja,Gurung,puja,Sales
Suman,Magar,suman,HR
```

`C:\Scripts\New-BulkUsers.ps1`
```powershell
$pw = Read-Host "Type first password" -AsSecureString
Import-Csv C:\Scripts\newusers.csv | ForEach-Object {
    if (Get-ADUser -Filter "SamAccountName -eq '$($_.Username)'") {
        "SKIP $($_.Username) - already exists"
    } else {
        New-ADUser -Name "$($_.First) $($_.Last)" -GivenName $_.First -Surname $_.Last `
            -SamAccountName $_.Username -UserPrincipalName "$($_.Username)@kumar.local" `
            -Department $_.Dept -Path "OU=Staff,OU=KumarCorp,DC=kumar,DC=local" `
            -AccountPassword $pw -Enabled $true -ChangePasswordAtLogon $true
        Add-ADGroupMember -Identity $_.Dept -Members $_.Username
        "Created $($_.Username) -> $($_.Dept)"
    }
}
```

| Part | Meaning |
|---|---|
| `Import-Csv ... \|` | read the file and pass each line on |
| `ForEach-Object { }` | do the same work for each user |
| `$_` | "this user" (the current line) |
| `Read-Host -AsSecureString` | type the first password safely (shows `****`) |
| `-ChangePasswordAtLogon $true` | the user picks their own password at first login |
| `if (...) { SKIP } else { create }` | don't create a user twice, and give an honest message |

Result: 6 users created in seconds, each in the right OU and group ✅

## 2. AD Recycle Bin – the "undo" button
```powershell
# check / turn on (one time, can't be turned off)
Get-ADOptionalFeature -Filter 'Name -like "Recycle Bin Feature"' | Select-Object Name, EnabledScopes
Enable-ADOptionalFeature -Identity "Recycle Bin Feature" -Scope ForestOrConfigurationSet -Target kumar.local

# find and restore a deleted user
Get-ADObject -Filter 'SamAccountName -eq "kiran"' -IncludeDeletedObjects | Select-Object Name, Deleted
Get-ADObject -Filter 'SamAccountName -eq "kiran"' -IncludeDeletedObjects | Restore-ADObject
```
Test: deleted Kiran → `ObjectNotFound` → restored → he came back **with his IT group** ✅

## 3. Windows Server Backup
Golden rule: **never keep the backup on the same disk as the data.**

```powershell
# on the laptop (Hyper-V host): add a 2nd disk to DC01
New-VHD -Path D:\VMs\Disks\DC01-Backup.vhdx -SizeBytes 40GB -Dynamic
Add-VMHardDiskDrive -VMName DC01 -Path D:\VMs\Disks\DC01-Backup.vhdx

# on DC01: bring it online and make E:
Set-Disk -Number 1 -IsOffline $false
Initialize-Disk -Number 1 -PartitionStyle GPT
New-Partition -DiskNumber 1 -UseMaximumSize -DriveLetter E | Format-Volume -FileSystem NTFS -NewFileSystemLabel Backup

# install and back up
Install-WindowsFeature Windows-Server-Backup
wbadmin start backup -backupTarget:E: -include:C:\Scripts -quiet
```
Test: deleted `important.txt` → **Recover** wizard → file back with the same date and size ✅

- New disks on a server start **Offline** (for safety).
- `NoChangeNeeded` = already installed, not an error.
- A backup you never tested is not a backup.

## 4. Month 2 Project – a new Finance department
📞 *"KumarCorp is opening a Finance department. 3 new staff start Monday. Set everything up!"*

| # | Task | How |
|---|---|---|
| 1 | Finance OU + Finance group | ADUC / `New-ADOrganizationalUnit`, `New-ADGroup` |
| 2 | 3 users from `finance.csv` | copied my script and changed the CSV name and OU |
| 3 | GPO: block Command Prompt | `Finance - Block CMD` linked to the Finance OU |
| 4 | Finance share | `\\dc01\Finance`, NTFS: Finance = Modify, Users removed |
| 5 | DNS nickname | CNAME `finance` → `dc01.kumar.local` |
| 6 | Tests | Ramesh: CMD blocked 🛡️, Budget.txt opens ✅ · Maya (HR): access denied 🚫 |
| 7 | Backup | `wbadmin start backup -backupTarget:E: -include:C:\Shares\Finance -quiet` ✅ |

Make a new script from an old one with one command (no Notepad):
```powershell
(Get-Content C:\Scripts\New-BulkUsers.ps1) -replace 'newusers.csv','finance.csv' -replace 'OU=Staff','OU=Finance' |
    Set-Content C:\Scripts\New-FinanceUsers.ps1
```

## Problems I solved

| Problem | Cause | Fix |
|---|---|---|
| Script made the old users again (SKIP anil, maya...) | Notepad changes **not saved**; PowerShell runs the file on disk | edited with `-replace` + `Set-Content`, then checked with `Get-Content \| Select-String` |
| Maya: "need the right to sign in through Remote Desktop Services" | Enhanced Session uses RDP | used **Basic Session** for normal users |
| `Add-VMHardDiskDrive`: "already connected" | I had already run it | checked with `Get-VMHardDiskDrive` ✅ |
| New disk was **Offline** | servers keep new disks offline for safety | Online → Initialize (GPT) → New Volume |
| Ramesh could still open CMD | GPO was linked and enabled but **empty** (setting not saved) | `gpresult /r /scope user` showed it missing → re-saved the setting → `gpupdate /force` → sign out/in ✅ |
| `gpupadate` not recognized | typo | read the red error, typed `gpupdate` |

## Key ideas
- Scripts are recipes: change the **ingredients** (file, OU), not the whole recipe.
- **Save, then check, then run.** PowerShell runs the file on disk.
- Red text has two kinds: ⌨️ typing mistake (`NamedParameterNotFound`) or 🧱 real problem (`already exists`, `ObjectNotFound`). Read the last words.
- Turn on the AD Recycle Bin **early**.
- Keep backups on a **separate disk**, and **test the restore**.
- `gpresult /r` shows which GPOs a user really got.

## In my own words
This week I stopped clicking one user at a time and wrote a PowerShell script that creates many users from a CSV file.
I learned to undo mistakes with the AD Recycle Bin and to back up and restore files with Windows Server Backup.
Then I built a whole Finance department by myself: OU, group, users, a GPO, a locked share, a DNS name, tests and a backup.
The hardest part was the GPO that did nothing. I used `gpresult` to find out it was empty, fixed it, and it worked. That felt like a real admin job.

**Month 2 (Windows Server & Active Directory) complete 🎓**
