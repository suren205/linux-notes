# Week 4 – Disks, LVM, Backups & Cron

## 1. Disk & memory health check
| Question | Command |
|---|---|
| Is the disk full? | `df -h` |
| What is filling it? | `sudo du -sh /var/* 2>/dev/null \| sort -h` |
| Which disks exist? | `lsblk` |
| Is RAM full? | `free -h` (look at `available`) |
| Journal size? | `journalctl --disk-usage` |
| Shrink journal safely | `sudo journalctl --vacuum-size=100M` |

**Real fix on my laptop:** found `/var/log/journal` at 500M using
`df → du → /var/log → journal`, cleaned it with `--vacuum-size`. Freed **401.7M**.

## 2. LVM (PV → VG → LV)
```bash
truncate -s 1G disk1.img                  # fake disk (lab only)
sudo losetup -fP --show disk1.img         # → /dev/loop0
sudo pvcreate /dev/loop0                  # PV: disk joins LVM
sudo vgcreate datavg /dev/loop0           # VG: the pool
sudo lvcreate -L 500M -n datalv datavg    # LV: the volume
sudo mkfs.ext4 /dev/datavg/datalv
sudo mount /dev/datavg/datalv /data
sudo lvextend -r -L +400M /dev/datavg/datalv   # grow online (-r = resize filesystem)
sudo vgextend datavg /dev/loop1                # add a new disk to the pool
```
Check: `pvs`, `vgs`, `lvs`
Cleanup order: `umount → lvremove → vgremove → pvremove → losetup -d → rm`

## 3. Backups
```bash
tar -czf box.tar.gz folder        # create
tar -tzf box.tar.gz               # list
tar -xzf box.tar.gz -C restore/   # extract into a test folder
diff -r restore/site site && echo SAME
rsync -av site/ mirror/           # mirror, copies only changes
```
> A backup you never tested is NOT a backup.

## 4. Cron
```
min hour day month weekday   command
0   2    *   *     *         /home/dell/scripts/backup.sh   # 2:00 AM daily
*/2 *    *   *     *         ...                            # every 2 minutes
```
`crontab -e` (edit), `crontab -l` (list), `#` = pause a job
Logs: `journalctl -u cron` → `CMD (...)` means cron ran it.

## Lab 1 – "Disk is full!" (incident report)
- **Problem:** `/data` at 96%, error `No space left on device`
- **Cause:** a 400M log file filled a 500M LV
- **Fix 1:** used free space in VG → `lvextend -r -L +400M` → 58%
- **Fix 2:** added a new disk → `vgextend` + `lvextend -r -L +500M` → 37%
- **Result:** fixed online, no downtime, no data lost

## Lab 2 – Automatic backup script
```bash
#!/bin/bash
# Back up the website and nginx settings
DEST=/home/dell/backups
mkdir -p "$DEST"
tar -czf "$DEST/web-$(date +%F-%H%M).tar.gz" /etc/nginx /var/www/html 2>/dev/null
echo "$(date) backup done" >> "$DEST/backup.log"
find /home/dell/backups -name "web-*.tar.gz" -mtime +7 -delete
```
- Scheduled with cron: `0 2 * * *`
- Restore test: `diff -r` → **RESTORE OK ✅**
- Old backups deleted after 7 days (`find -print` first, then `-delete`)

## Bandit
- Level 8 → 9: `sort data.txt | uniq -u`
- Level 9 → 10: `strings data.txt | grep "=="`

## What I learned
- Always check → fix → verify.
- Never `rm` logs blindly; use the official tool.
- Variables in a script live only inside that script.
