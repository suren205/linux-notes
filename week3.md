# Linux Week 3 – Networking, SSH Keys and Firewall

Notes from my third week of learning Linux (Ubuntu 24.04 on WSL).

## 1. IP addresses

| Command | What it does |
|---|---|
| `ip -br addr` | Shows my IP addresses (short) |
| `ip route` | Shows the gateway (`default via ...`) |
| `ping -c 4 8.8.8.8` | Tests if a computer answers, and how fast |

- `127.0.0.1` = localhost (this computer). `::1` is the same in IPv6
- Private addresses: `10.x`, `172.16–31.x`, `192.168.x`
- `172.217.x.x` starts with 172 but is public (217 is not 16–31)

How I test a network problem, from close to far:
1. `ping 127.0.0.1` (myself)
2. `ping` the gateway
3. `ping 8.8.8.8` (internet by number)
4. `ping google.com` (internet by name)

If 3 works but 4 fails, the problem is DNS.

## 2. Ports and DNS

| Port | Service |
|---|---|
| 22 | SSH |
| 53 | DNS |
| 80 | HTTP |
| 443 | HTTPS |

| Command | What it does |
|---|---|
| `sudo ss -tulpn` | Shows open ports and which program uses them |
| `sudo ss -tulpn \| grep :80` | Checks one port |
| `dig +short google.com` | Asks DNS for the IP of a name |
| `cat /etc/resolv.conf` | Shows which DNS server my computer asks |
| `/etc/hosts` | My own small list of names, read before DNS |

Big websites have many IP addresses, and the answer can change.

## 3. Lab: network detective

| Question | My answer |
|---|---|
| IP of eth0 | 172.29.220.25 (private) |
| Gateway | 172.29.208.1 (Windows, on WSL) |
| Ping test | All 4 steps worked, 0% loss. ~0.05 ms → 0.8 ms → ~8 ms |
| DNS server | 10.255.255.254 (WSL helper) |
| amazon.com | 3 IP addresses |
| Port 80 | nginx (1 master + 16 workers, IPv4 and IPv6) |
| Own name | Added `127.0.0.1 kumar.local` to /etc/hosts (backup first) |
| nginx stopped | `ss` shows nothing on port 80 |

## 4. SSH keys

| File | Meaning |
|---|---|
| `id_ed25519` | Private key. Never share. Permission 600 |
| `id_ed25519.pub` | Public key. Safe to share |
| `authorized_keys` | Public keys a server accepts |

| Command | What it does |
|---|---|
| `ssh-keygen -t ed25519 -C "kumar laptop"` | Makes a key pair |
| `ssh-copy-id dell@localhost` | Puts my public key on a server |
| `ssh dell@localhost` | Logs in with the key, no password |
| `ssh -T git@github.com` | Tests my key on GitHub |

I added my public key to GitHub and got:
"You've successfully authenticated".

## 5. Firewall (ufw)

| Command | What it does |
|---|---|
| `sudo apt install ufw -y` | Installs ufw (not included in WSL Ubuntu) |
| `sudo ufw allow 22/tcp` | Opens SSH |
| `sudo ufw allow 80/tcp` | Opens the web port |
| `sudo ufw enable` | Turns the firewall on |
| `sudo ufw status numbered` | Lists rules with numbers |
| `sudo ufw delete allow 443/tcp` | Deletes a rule by name (safer) |

Golden rule: always allow port 22 BEFORE `ufw enable` on a remote server.

## 6. Incident report: key login stopped working

- Sign: `Permission denied (publickey)`
- How I found it: `sudo journalctl -u ssh -n 30 --no-pager | grep -i "bad"`
  showed `bad ownership or modes for file /home/dell/.ssh/authorized_keys`
- Cause: the file had permission 777, so SSH refused to trust it
- Fix: `chmod 600 ~/.ssh/authorized_keys`, then tested the key login
- Lesson: SSH refuses keys when files are not private

## 7. Mistakes I made and what I learned

- I deleted the wrong firewall rule (SSH for IPv6) because the rule
  numbers changed. Now I read the "Deleting:" line before I type y
- `ufw` was "command not found" because it was not installed. I installed it with apt
- My GitHub key was "invalid" for 2 hours. The key was fine.
  The problem was copy and paste. Lesson: check the input first
- `curl http://kumar.loca` gave "Could not resolve host". That is a name (DNS) error

## 8. Other practice

- OverTheWire Bandit levels 6 and 7: `find / -user -group -size 2>/dev/null`, `grep`
- `2>/dev/null` hides error messages
