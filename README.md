# 🐧 Linux Notes – System Engineer Journey

I am learning to become a **System Engineer**. This repo has my notes and hands-on labs.
Everything was practised on **Ubuntu 24.04 (WSL)**.

## ✅ Month 1 – Linux (completed)

| Week | Topic | Notes |
|---|---|---|
| 1 | Files, folders, permissions, users & groups | [week1.md](week1.md) |
| 2 | Packages, services (systemd), nginx, logs, troubleshooting | [week2.md](week2.md) |
| 3 | Networking, DNS, SSH keys, firewall (ufw) | [week3.md](week3.md) |
| 4 | Disks, LVM, backups (tar/rsync), cron automation | [week4.md](week4.md) |

## 🛠️ Skills
- Linux command line, file permissions (`chmod`, `chown`), users and groups
- Installing and managing services with `apt` and `systemctl`
- Web server setup and troubleshooting (nginx, `curl`, `journalctl`)
- Networking checks (`ip`, `ping`, `ss`, `dig`) and firewall rules (`ufw`)
- SSH key login and GitHub SSH
- Disk checks (`df`, `du`, `lsblk`, `free`) and LVM (grow a disk with no downtime)
- Backup scripts in Bash, scheduled with `cron`, with tested restores

## 🧪 Labs I completed
- **Broken web server** – found and fixed nginx problems using logs (5-step troubleshooting)
- **Disk full** – fixed a full disk online with `lvextend` and `vgextend`
- **Real cleanup** – freed 401 MB of old system logs on my own laptop
- **Backup robot** – Bash script + cron at 2 AM + restore test (`diff -r`) + auto-delete after 7 days
- **OverTheWire Bandit** – levels 0–9

## 🗺️ Roadmap
- [x] Month 1 – Linux
- [ ] Month 2 – Windows Server & Active Directory
- [ ] Month 3 – Virtualization & Storage
- [ ] Month 4 – AWS
- [ ] Month 5 – Bash & Ansible automation
- [ ] Month 6 – Portfolio & job hunt
