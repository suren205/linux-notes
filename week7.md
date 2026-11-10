# Week 7 – DHCP, DNS Records & Troubleshooting

## Lab
- **DC01** – Windows Server 2025, Domain Controller + DNS + **DHCP** for `kumar.local` (192.168.50.10)
- **CLIENT01** – Windows 11 Enterprise, joined to the domain, now gets its IP from DHCP
- Network: internal switch `LabSwitch` + NAT `LabNAT` (192.168.50.0/24, gateway 192.168.50.1 = host)

## 1. DHCP – automatic IP addresses
DHCP gives every PC an IP, gateway and DNS server, so I don't type them by hand.

Install and authorize on DC01:
```powershell
Install-WindowsFeature DHCP -IncludeManagementTools
Add-DhcpServerInDC -DnsName dc01.kumar.local -IPAddress 192.168.50.10
Add-DhcpServerSecurityGroup
Restart-Service DHCPServer
```

| Scope setting | Value |
|---|---|
| Range | 192.168.50.100 – 192.168.50.200 |
| Lease | 8 days |
| Router (gateway) | 192.168.50.1 |
| DNS server | 192.168.50.10 |
| DNS domain | kumar.local |

- In a domain, a DHCP server must be **authorized in AD**, or it will not give out addresses.
- CLIENT01 got **192.168.50.100** 

On the client:
```powershell
ipconfig /all       # see IP, gateway, DNS, DHCP server
ipconfig /release   # give the IP back
ipconfig /renew     # ask DHCP for a new IP
```

## 2. DNS records
DNS is the "phone book": it turns names into numbers.

| Record | Direction | Example |
|---|---|---|
| **A** | name → IP | `dc01` → 192.168.50.10 |
| **CNAME** | nickname → real name | `intranet` → `dc01.kumar.local` |
| **PTR** | IP → name (reverse) | 192.168.50.10 → `dc01` |

- PTR records live in a **reverse lookup zone**: `50.168.192.in-addr.arpa`
- Test with:
```powershell
nslookup dc01
nslookup intranet
nslookup 192.168.50.10
ipconfig /flushdns   # clear old answers from the cache
```

## 3. Troubleshooting ladder 
Go from **near → far** and **simple → deep**: *Me → Door → Server → Names → Logs*

| Step | Check | Command |
|---|---|---|
| 1 | My IP | `ipconfig` |
| 2 | Gateway (the door) | `ping 192.168.50.1` |
| 3 | DC by number | `ping 192.168.50.10` |
| 4 | DNS (names) | `nslookup dc01` |
| 5 | Service + logs | `Get-Service`, Event Viewer (`eventvwr.msc`) |

## 4. Help desk tickets (I broke it, then fixed it)

| Ticket | User says | Clue | Cause | Fix |
|---|---|---|---|---|
| #1 | "No network!" | IP = **169.254.x.x** (APIPA) | DHCP service stopped | `Start-Service DHCPServer`, then `ipconfig /renew` |
| #2 | "Number works, name fails" | `ping 192.168.50.10` , `ping dc01` | DNS set to 1.2.3.4 | DNS back to 192.168.50.10, `ipconfig /flushdns` |
| #3 | "I can't log in!" | "Your account has been disabled" | Account disabled | `Enable-ADAccount -Identity ram` |

Check an account:
```powershell
Get-ADUser -Identity ram | Select-Object Name, Enabled
Disable-ADAccount -Identity ram
Enable-ADAccount -Identity ram
```

## 5. Real incident: DC01 "Paused-Critical" 
**What happened:** DC01 suddenly showed *Paused-Critical* in Hyper-V Manager.

**Cause:** drive D: had **0 GB free**. I had **17 checkpoint files** (`.avhdx`) that kept growing.
When a VM can't write to its disk, Hyper-V pauses it to protect the data.

**Fix:**
1. Ejected the ISOs and deleted the unused Win11 ISO → 7.5 GB free
2. `Resume-VM -Name DC01` → DC01 running again
3. Moved the Server ISO to C:
4. Shut down both VMs and removed all checkpoints (Hyper-V merged them)

```powershell
Get-Volume -DriveLetter C, D
Get-VMSnapshot -VMName DC01 | Remove-VMSnapshot
while ((Get-VM DC01).Status -like "*Merg*") { Get-Date -Format T; Start-Sleep 30 }; "DONE"
```
**Result:** D: back to **99.7 GB free** 

**New rule:** keep only **1–2 checkpoints**, delete them after testing, and check free space often.

## Problems I solved

| Problem | Fix |
|---|---|
| Client had no internet after DHCP | gateway typo in the scope (`192.167.50.1`) → `Set-DhcpServerv4OptionValue -ScopeId 192.168.50.0 -Router 192.168.50.1` |
| Client still used old static DNS | `Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ResetServerAddresses` |
| Ram could still log in after "disable" | the command had not worked → checked `Enabled`, ran it again on DC01, signed out (not just locked) |
| DC01 Paused-Critical | disk full from checkpoints (see section 5) |

## Key ideas
- **169.254.x.x = DHCP problem.** Number works but name fails = **DNS problem.**
- Always **check first, then fix**: read the error message, look at `Enabled`, look at `ipconfig`.
- Run `hostname` before every command, so I know which machine I'm on.
- PowerShell is **Verb-Noun**: `Get-` to look, `Enable-` / `Disable-` / `Set-` to change.
- Watch disk space. Too many checkpoints can stop a server.

## In my own words
This week I made DC01 give out IP addresses with DHCP and added DNS records like A, CNAME and PTR.
Then I worked like a help desk engineer: I broke the network three times and fixed it using the troubleshooting ladder.
I also had a real problem: my DC paused because the disk was full of checkpoints. I found the cause and fixed it.
The best lesson: don't guess. Check the IP, the DNS, and the error message first.
