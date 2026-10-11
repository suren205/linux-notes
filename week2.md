# Linux Week 2 – Packages, Services, Logs and Troubleshooting

Notes from my second week of learning Linux (Ubuntu 24.04 on WSL).
This week I built a real web server with Nginx, then broke it and fixed it.

## 1. Installing software (apt)

| Command | What it does |
|---|---|
| `sudo apt update` | Refreshes the list of software. Installs nothing |
| `sudo apt upgrade -y` | Installs the newer versions |
| `apt show htop` | Shows information about a package |
| `sudo apt install nginx -y` | Installs a package |
| `dpkg -l \| grep nginx` | Checks if a package is installed |
| `which htop` | Shows where a program lives |

Other Linux systems (Red Hat, Rocky, Amazon Linux) use `dnf` instead of `apt`.

The pipe `|` sends the output of one command into the next command.

## 2. Services (systemctl)

| Command | What it does |
|---|---|
| `systemctl status nginx` | Full status of a service |
| `systemctl is-active nginx` | Running now? (active / inactive) |
| `systemctl is-enabled nginx` | Starts at boot? (enabled / disabled) |
| `sudo systemctl start nginx` / `stop` | Start or stop now |
| `sudo systemctl restart nginx` | Stop and start again |
| `sudo systemctl enable nginx` / `disable` | Start at boot or not |
| `curl -I http://localhost` | Test a website (200 OK) |

Rule: start = now, enable = at the next boot. Always do both.

## 3. Logs

| Command | What it does |
|---|---|
| `sudo journalctl -u nginx -n 20 --no-pager` | Last 20 log lines of a service |
| `sudo tail -n 5 /var/log/nginx/access.log` | Last visits to the website |
| `sudo tail -f /var/log/nginx/access.log` | Watch visits live (Ctrl+C to stop) |
| `sudo grep -c " 404 " /var/log/nginx/access.log` | Count "not found" visits |

Status codes: 200 = OK, 404 = not found, 500 = server error.
4xx is usually the visitor's mistake. 5xx is our problem.

I found 8 "404" lines but only visited 2 bad pages. The log showed the
other 6 were Chrome asking for `/favicon.ico`. Logs show what really happened.

## 4. How I fix problems (5 steps)

1. Status: `systemctl status nginx`
2. Logs: `sudo journalctl -u nginx -n 20 --no-pager`
3. Config test: `sudo nginx -t`
4. Fix one thing only: `sudo nano <file>`
5. Restart and test: `sudo systemctl restart nginx` then `curl -I http://localhost`

Before changing a settings file: `sudo cp file file.bak`, then check it exists with `ls -l`.

## 5. Lab: my own web server

- Installed Nginx and checked it with `systemctl` and `curl`
- Made my own page with `echo "..." | sudo tee /var/www/html/index.html`
- Opened it in my Windows browser at `http://localhost`

## 6. Lab: incident reports (break and fix)

**Case A – Website down**
- Sign: `curl` said "Failed to connect"
- Cause: the service was stopped (`inactive`)
- Fix: `sudo systemctl start nginx`, tested with `curl` (200 OK)

**Case B – Nginx would not start after a settings change**
- Sign: `restart` failed, website down
- Cause: a bad line at line 85 of `/etc/nginx/nginx.conf`
- How I found it: `journalctl` and `nginx -t` both showed `nginx.conf:85`
- Fix: deleted the line with `nano`, ran `nginx -t`, restarted, tested
- Lesson: my backup failed because I typed `sodu` instead of `sudo`.
  Now I always check that the backup file exists.

**Case C – Hidden problem**
- Sign: none, the website worked
- Cause: the service was `disabled`, so it would not start after a reboot
- Fix: `sudo systemctl enable nginx`, checked with `is-enabled`

## 7. Mistakes I made and what I learned

- `-l` (letter L) is not `-1` (number one). The long form `--list` avoids this
- `systemctl start nginx` needs a space, not a dash
- Use the Tab key to finish words and avoid typos
- `ssh` to the Bandit game needs `-p 2220` (the port).
   My ssh login to Bandit failed. The error said "port 22 is not intended". I added `-p 2220` and it worked. Now I read error messages first.
- `cat ./*` prints every file.
  I typed `cat ./*` and my screen filled with strange symbols, because it printed all 10 files. I fixed the screen with `reset`.
- Never delete a file to fix a problem. Look first
- Never paste passwords into a chat

## 8. Other practice

- Finished OverTheWire Bandit levels 4 and 5 (`file`, `find -size`)
