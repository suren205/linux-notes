# Week 5 – Windows Server, Active Directory & Domain Join

## Lab setup
- Host: Windows 11 Pro laptop with **Hyper-V** (VMs stored on D:)
- **DC01** – Windows Server 2025 (Desktop Experience), 2 GB RAM, IP `192.168.50.10`
- **CLIENT01** – Windows 11 Enterprise, Dynamic Memory (1–4 GB), IP `192.168.50.20`
- Lab network: internal switch **LabSwitch** + **NAT** (laptop = router `192.168.50.1`)

```
Internet
   │
Laptop (Hyper-V host) ── 192.168.50.1 (NAT)
   │
LabSwitch
   ├── DC01      192.168.50.10  (Domain Controller + DNS)
   └── CLIENT01  192.168.50.20  (joined to kumar.local)
```

## 1. Hyper-V (run on the host)
```powershell
New-VM -Name DC01 -Generation 2 -MemoryStartupBytes 4GB -NewVHDPath D:\VMs\Disks\DC01.vhdx -NewVHDSizeBytes 60GB -SwitchName "Default Switch"
New-VMSwitch -SwitchName LabSwitch -SwitchType Internal
New-NetIPAddress -IPAddress 192.168.50.1 -PrefixLength 24 -InterfaceAlias "vEthernet (LabSwitch)"
New-NetNat -Name LabNAT -InternalIPInterfaceAddressPrefix 192.168.50.0/24
Checkpoint-VM -Name DC01 -SnapshotName "AD ready - kumar.local"
Set-VMMemory -VMName CLIENT01 -DynamicMemoryEnabled $true -StartupBytes 2GB -MinimumBytes 1GB -MaximumBytes 4GB
```

## 2. Server setup (inside DC01)
```powershell
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.50.10 -PrefixLength 24 -DefaultGateway 192.168.50.1
Rename-Computer -NewName DC01 -Restart
Set-TimeZone -Id "Tokyo Standard Time"
```

## 3. Active Directory
```powershell
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
Install-ADDSForest -DomainName "kumar.local" -DomainNetbiosName "KUMAR" -InstallDns
```
Structure:
```
kumar.local
  KumarCorp
    Staff  -> Ram Sharma (IT), Sita Rai (HR), Hari Thapa (Sales)
    Groups -> IT, HR, Sales
```
```powershell
New-ADUser -Name "Sita Rai" -SamAccountName sita -Path "OU=Staff,OU=KumarCorp,DC=kumar,DC=local" -AccountPassword $pw -Enabled $true
New-ADGroup -Name "HR" -GroupScope Global -GroupCategory Security -Path "OU=Groups,OU=KumarCorp,DC=kumar,DC=local"
Add-ADGroupMember -Identity "HR" -Members sita
Set-ADAccountPassword -Identity ram -Reset -NewPassword (Read-Host -AsSecureString)
Disable-ADAccount -Identity sita
Enable-ADAccount -Identity sita
```
I also did the same tasks in the GUI (Active Directory Users and Computers).

## 4. Domain join (inside CLIENT01, admin terminal)
```powershell
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.50.20 -PrefixLength 24 -DefaultGateway 192.168.50.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 192.168.50.10
Resolve-DnsName kumar.local
Add-Computer -DomainName kumar.local -Credential KUMAR\Administrator -Restart
```
Result: logged in to CLIENT01 as **KUMAR\ram** ✅

## Problems I solved
| Problem | Cause | Fix |
|---|---|---|
| No desktop after install | chose Server Core | reinstalled with Desktop Experience |
| VM would not boot the installer | DVD drive was empty | `Set-VMDvdDrive` with the ISO path |
| "Access is denied" | terminal not admin (UAC) | Run as administrator |
| Command ran on the wrong machine | same prompt on host and VM | always check `hostname` first |
| "Insufficient system resources" | not enough host RAM | `wsl --shutdown`, smaller VM, Dynamic Memory |
| "The Ethernet switch may not exist" | broken VM network link after a crash | disconnect and reconnect the network adapter |
| Join OK but rename failed | "directory service is busy" | renamed again separately |
| Kerberos risk | DC time zone was wrong | set Tokyo time zone |

## Key ideas
- A **Domain Controller** is the main server that stores all user accounts and checks every login.
- Clients use the **DC as their DNS server**, so they can find the domain.
- A user's account lives **on the DC**; the client asks the DC to check the password.
- **Disable** accounts for leave; don't delete them.
- Windows 11 **Home cannot join a domain** (Pro / Enterprise / Education can).

## In my own words
This week I built my own small company network on my laptop.
I made a Domain Controller, created users and groups, and joined a Windows 11 PC to the domain.
The best moment was logging in as Ram on CLIENT01 with an account that only exists on DC01.
I also learned to stop and check facts when something fails again and again.
