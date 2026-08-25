# Day 2 — Linux Filesystem Hierarchy

## Goal
Get comfortable navigating a Linux system and understand what each top-level directory is for.

## Environment
Practiced on KillerCoda (browser-based Linux terminal).

## Commands used
```bash
ls -la /
cd /etc && ls
cd /var && ls
cd /proc && ls
cd /home && ls
cd /usr && ls
man hier
df -h
du -sh /var/*
```

## Key directories explored

| Directory | Purpose |
|---|---|
| `/` | Root of the entire filesystem — everything branches from here |
| `/etc` | System-wide configuration files (e.g. `/etc/passwd`, `/etc/hosts`, service configs) |
| `/var` | Variable data — logs (`/var/log`), cache, spool, mail. Grows over time, needs monitoring |
| `/proc` | Virtual filesystem — doesn't exist on disk. Exposes live kernel/process info (e.g. `/proc/cpuinfo`, `/proc/<pid>`) |
| `/home` | Personal directories for each user, e.g. `/home/nitin` |
| `/usr` | User-installed programs, libraries, and shared resources — most of the actual "system" lives here |
| `/bin` , `/sbin` | Essential binaries needed for the system to boot/run (ls, cp, mount, etc.) |
| `/tmp` | Temporary files, cleared on reboot |

## Useful commands learned
- `man hier` — manual page that documents the entire filesystem hierarchy standard (FHS)
- `df -h` — shows disk usage per mounted filesystem, human-readable
- `du -sh /var/*` — shows size of each item inside `/var`, useful for finding what's eating disk space

## Notes / takeaways
- `/etc` and `/var` are the two directories I'll be touching most often as a DevOps engineer — configs live in one, logs live in the other.
- `/proc` is not real files on disk, it's the kernel exposing live system state — good to know before I get into process management (Day 5).
- Filesystem layout is standardized (FHS) across most Linux distros, so this maps to any server I'll work on later — not just this sandbox.

## Status
✅ Day 2 complete — filesystem hierarchy explored and documented.

# Day 3 — File Permissions & Ownership

## Goal
Understand Linux permission strings, and practice chmod, chown, chgrp, and umask.

## Environment
Practiced on KillerCoda.

## Commands used
```bash
ls -la
touch testfile.sh
chmod 755 testfile.sh
chmod u+x testfile.sh
chmod go-w testfile.sh
sudo useradd testuser
sudo chown testuser testfile.sh
sudo chgrp testuser testfile.sh
umask
touch newfile.txt
```

## Key concepts

**Permission string breakdown** (e.g. `-rwxr-xr-x`):
- 1st char: file type (`-` = file, `d` = directory, `l` = symlink)
- Next 3: owner permissions (rwx)
- Next 3: group permissions (r-x)
- Last 3: others permissions (r-x)

**chmod — two ways to set permissions**
- Numeric: `chmod 755 file` → owner=rwx(7), group=rx(5), others=rx(5)
- Symbolic: `chmod u+x file` (add execute for owner), `chmod go-w file` (remove write for group+others)

| Number | Permission |
|---|---|
| 7 | rwx (read+write+execute) |
| 6 | rw- (read+write) |
| 5 | r-x (read+execute) |
| 4 | r-- (read only) |

**chown / chgrp**
- `chown user file` — changes file owner
- `chgrp group file` — changes group owner
- Can combine: `chown user:group file`

**umask**
- Determines default permissions for newly created files/directories
- Default umask (often `022`) subtracts write permission for group/others → new files typically land at `644`, new directories at `755`

## Mini task — fix-perms.sh
Wrote a script that normalizes permissions inside a given folder:
```bash
#!/bin/bash
# Usage: ./fix-perms.sh <directory>
find "$1" -type f -exec chmod 644 {} \;
find "$1" -type d -exec chmod 755 {} \;
echo "Permissions normalized for $1"
```
This mirrors a real DevOps scenario — fixing permission drift after a deploy or file transfer where permissions get messed up.

## Notes / takeaways
- Permissions are one of the most common sources of "it works on my machine but not on the server" bugs — worth being precise here.
- `find ... -exec chmod` is the pattern I'll reuse a lot for bulk permission fixes.
- My security background made the permission model intuitive — this is the same least-privilege thinking from bug bounty work, just applied to a filesystem instead of an app.

## Status
✅ Day 3 complete — permissions, ownership, and umask practiced; wrote fix-perms.sh
