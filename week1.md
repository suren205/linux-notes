# Linux Week 1 – Files, Permissions, Users and Groups

Notes from my first week of learning Linux (Ubuntu on WSL).

## 1. Moving around

| Command | What it does |
|---|---|
| `pwd` | Shows the folder I am in now |
| `ls -la` | Lists all files with details, including hidden files |
| `cd /etc` | Goes to a folder |
| `cd ..` | Goes up one folder |
| `cd ~` | Goes to my home folder |

Important folders: `/etc` (settings), `/var/log` (logs), `/home` (user files), `/tmp` (temporary files).

## 2. Files and folders

| Command | What it does |
|---|---|
| `mkdir -p a/b/c` | Makes folders, including parent folders |
| `touch file.txt` | Makes an empty file |
| `echo "text" > file.txt` | Writes text to a file (replaces the old content) |
| `echo "text" >> file.txt` | Adds text to the end of a file |
| `cp -r lab lab-backup` | Copies a folder and everything inside |
| `mv old.txt new.txt` | Moves or renames a file |
| `rm -r folder` | Deletes a folder forever (no Recycle Bin) |
| `cat`, `less`, `head`, `tail` | Read files |
| `find . -name "*.txt"` | Finds files by name |
| `grep -r "word" folder` | Finds text inside files |

## 3. Permissions

Example: `-rw-r--r-- 1 dell dell 15 note.txt`

- First letter: `-` = file, `d` = folder
- Then 3 blocks: owner, group, others
- `r` = read (4), `w` = write (2), `x` = execute (1)

| Number | Letters | Used for |
|---|---|---|
| 644 | `rw-r--r--` | normal files |
| 755 | `rwxr-xr-x` | scripts and folders |
| 600 | `rw-------` | secret files |
| 770 | `rwxrwx---` | team folders |

Commands: `chmod 755 script.sh`, `chmod u+x script.sh`, `sudo chown root:sales /srv/sales`

## 4. Users and groups

| Command | What it does |
|---|---|
| `whoami`, `id` | Shows my user and my groups |
| `sudo useradd -m -s /bin/bash aiko` | Makes a new user with a home folder |
| `sudo passwd aiko` | Sets a password |
| `sudo groupadd sales` | Makes a new group |
| `sudo usermod -aG sales aiko` | Adds a user to a group (keeps other groups) |
| `sudo su - aiko` | Switches to another user |
| `sudo userdel -r aiko` | Deletes a user and the home folder |

## 5. Lab: shared team folder

Task: make a folder that only the sales team can use.

```bash
sudo groupadd sales
sudo mkdir -p /srv/sales
sudo chown root:sales /srv/sales
sudo chmod 770 /srv/sales
sudo usermod -aG sales aiko
```

Test result: a user in the `sales` group could create files. A user outside the group got "Permission denied".

## 6. Mistakes I made and what I learned

About cd ~
- I typed `cd~` many times and got "command not found". Now I remember: a command and its target always need a space between them.
- `ls -1` (number one) is not the same as `ls -l` (letter L)
About > and >>
- I put three `echo` commands on one line and my jan.txt became empty. I learned that `>` empties the file first, so I must type one command per line and check with `cat`.
- A folder needs `x` to enter it with `cd`
- "Permission denied" on a script is fixed with `chmod +x`, not with `rm`
About the prompt
- I ran `cat ./-` on my own laptop instead of the Bandit server and got "No such file". Now I always read the prompt first: `dell@SUREN` is my laptop, `bandit1@bandit` is the server.
About sudo
- I tried `sudo` while I was logged in as the test user ravi, and it failed. Only an admin can add a user to a group. A user cannot add himself.
About the Windows folder
- Ubuntu first opened in /mnt/c, which is my Windows C: drive. I added `cd ~` to my .bashrc so I always start in my Linux home and keep my Windows files safe.
- Never share passwords

## 7. Other practice

- Connected to a remote server with `ssh`
- Finished OverTheWire Bandit levels 0–3
