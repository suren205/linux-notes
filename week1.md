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

- `cd~` does not work. I need a space: `cd ~`
- `ls -1` (number one) is not the same as `ls -l` (letter L)
- `>` replaces a file. I emptied a file by mistake. `>>` adds to the end
- A folder needs `x` to enter it with `cd`
- "Permission denied" on a script is fixed with `chmod +x`, not with `rm`
- Always check the prompt to know which user and which computer I am on
- Never share passwords

## 7. Other practice

- Connected to a remote server with `ssh`
- Finished OverTheWire Bandit levels 0–3
