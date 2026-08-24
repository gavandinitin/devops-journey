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
