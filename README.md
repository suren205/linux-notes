# 🐧🪟 System Engineer Journey – Notes & Labs

I am learning to become a **System Engineer**. This repo has my notes and hands-on labs.
Linux was practised on **Ubuntu 24.04 (WSL)**; Windows labs run on **Hyper-V** on my laptop.

## ✅ Month 1 – Linux (completed)

| Week | Topic | Notes |
|---|---|---|
| 1 | Files, folders, permissions, users & groups | [week1.md](week1.md) |
| 2 | Packages, services (systemd), nginx, logs, troubleshooting | [week2.md](week2.md) |
| 3 | Networking, DNS, SSH keys, firewall (ufw) | [week3.md](week3.md) |
| 4 | Disks, LVM, backups (tar/rsync), cron automation | [week4.md](week4.md) |

## 🔄 Month 2 – Windows Server & Active Directory (in progress)

| Week | Topic | Notes |
|---|---|---|
| 5 | Hyper-V lab, Windows Server 2025, Active Directory, users & groups, domain join | [week5.md](week5.md) |
| 6 | Group Policy, file shares & NTFS permissions, RSAT | [week6.md](week6.md) |

## 🛠️ Skills
**Linux**
- Command line, file permissions (`chmod`, `chown`), users and groups
- Services with `apt` and `systemctl`; nginx troubleshooting (`curl`, `journalctl`)
- Networking (`ip`, `ping`, `ss`, `dig`), SSH keys, firewall (`ufw`)
- Disks (`df`, `du`, `lsblk`, `free`), LVM, Bash backup scripts with `cron`

**Windows Server**
- Hyper-V: VMs, virtual switches, NAT, checkpoints, Dynamic Memory
- Windows Server 2025 setup: fixed IP, DNS, time zone
- Active Directory: domain, OUs, users, security groups (GUI and PowerShell)
- Help desk tasks: password reset, disable / enable accounts
- Joining a Windows 11 client to a domain

## 🧪 Labs I completed
- **Broken web server** – found and fixed nginx problems using logs
- **Disk full** – fixed a full disk online with `lvextend` and `vgextend`
- **Backup robot** – Bash script + cron + tested restore
- **OverTheWire Bandit** – levels 0–9
- **Company domain lab** – built `kumar.local` with a Domain Controller and a Windows 11 client; logged in on the client with a domain account

## 🗺️ Roadmap
- [x] Month 1 – Linux
- [ ] Month 2 – Windows Server & Active Directory (in progress)
- [ ] Month 3 – Virtualization & Storage
- [ ] Month 4 – AWS
- [ ] Month 5 – Bash & Ansible automation
- [ ] Month 6 – Portfolio & job hunt
