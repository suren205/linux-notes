# Week 6 – Group Policy, File Shares & Remote Administration

## Lab
- **DC01** – Windows Server 2025, Domain Controller for `kumar.local`
- **CLIENT01** – Windows 11 Enterprise, joined to the domain
- OUs: `KumarCorp → Staff, Groups`; groups: IT, HR, Sales

## 1. Group Policy (GPO)
A GPO is a "rule book" made on the DC and applied to every computer or user in an OU.

Steps (Group Policy Management on DC01):
1. Right-click the **Staff** OU → *Create a GPO in this domain, and Link it here* → `Block Control Panel`
2. Edit → **User Configuration → Policies → Administrative Templates → Control Panel**
3. *Prohibit access to Control Panel and PC settings* → **Enabled**

Test on CLIENT01 as `KUMAR\ram`:
```powershell
gpupdate /force   # get the new rules now
gpresult /r       # show which GPOs were applied
```
Result: opening Control Panel → *"This operation has been cancelled due to restrictions"* ✅

- **User Configuration** follows the person; **Computer Configuration** follows the machine.
- To undo a GPO: uncheck **Link Enabled** (the GPO stays saved).

## 2. File share with permissions
Goal: an `HR` folder that **only the HR group** can open.

```
C:\Shares\HR  →  shared as \\DC01\HR
```

| Lock | Setting |
|---|---|
| Share permission (the door) | Everyone: Full Control |
| NTFS permission (the real lock) | inheritance disabled, **Users removed**, **HR: Modify**, Administrators + SYSTEM: Full Control |

Tests on CLIENT01:

| User | Group | `\\DC01\HR` |
|---|---|---|
| Sita | HR | ✅ opens and edits `Salary.txt` |
| Ram | IT | 🚫 "You do not have permission to access \\DC01\HR" |

## 3. Remote administration with RSAT
Real admins don't sit at the Domain Controller. They manage AD from their own PC.

On CLIENT01, logged in as `KUMAR\Administrator`:
```powershell
Add-WindowsCapability -Online -Name Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0
```
Then **Win + R → `dsa.msc`** opens Active Directory Users and Computers on the client.

Test: created user **Gita Karki** and added her to **IT** from CLIENT01 → she appeared on DC01 ✅

## Problems I solved

| Problem | Fix |
|---|---|
| Ram couldn't log in through Enhanced Session ("need the right to sign in through Remote Desktop Services") | switched the VM window to **basic session** (Enhanced Session uses RDP) |
| Shared the wrong folder (`Shares` instead of `HR`) | unshared it; always check the **window title** before changing settings |
| Simple "Share..." wizard changes both locks | used **Advanced Sharing** instead |
| RSAT install was very slow | it downloads from Windows Update; waited for `Online : True` |

## Key ideas
- Give permissions to **groups**, not to single users. New HR person → add to the HR group → access works.
- **Least privilege**: give only what people need (Modify, not Full Control).
- Always **test both sides**: the right person gets in, the wrong person stays out.
- `gpupdate /force` → get rules now; `gpresult /r` → see which rules you got.
- Admins manage AD remotely with **RSAT** (`dsa.msc`), not by sitting at the DC.

## In my own words
This week I learned to control many computers from one place with Group Policy.
I made a secret HR folder that only the HR team can open, and I tested it with two users.
I also installed RSAT on my client PC and created a new user from there, like a real IT admin.
The best part was seeing the "Access denied" message for Ram, because it proved my lock works.
