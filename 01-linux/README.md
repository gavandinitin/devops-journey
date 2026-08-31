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

# Day 4 — Process Management

## Goal
Learn to view, control, and prioritize processes on Linux.

## Environment
Practiced on KillerCoda.

## Commands used
```bash
ps aux
sleep 300 &
jobs
kill -10 1027
nice -n 10 sleep 100 &
renice 5 -p 1079
sudo renice 5 -p 1079
killall sleep
nano find-hog.sh
chmod +x find-hog.sh
./find-hog.sh
```

## What happened (real troubleshooting)

**Killing a background job**
- Started `sleep 300 &`, tried `kill -10 <PID>`
- Result: `User defined signal 1` — NOT actually killed. `-10` is `SIGUSR1`, a signal meant for custom app-defined behavior, not termination.
- Lesson: signal numbers matter. `kill -9` (SIGKILL) or plain `kill` (SIGTERM) actually terminate a process; `kill -10` does something else entirely depending on what the process does with it.

**renice — process ownership matters**
- `renice 5 -p 1027` → failed: `No such process` (process had already ended/wasn't right PID)
- `renice 5 -p 1079` → failed: `Permission denied` (my user doesn't own the priority-lowering privilege by default)
- `sudo renice 5 -p 1079` → worked: `old priority 10, new priority 5`
- Lesson: lowering a process's niceness (making it higher priority) requires root, even if you own the process. Regular users can only make their own processes *less* favorable (higher nice value), not more.

**killall**
- `killall sleep` cleanly terminated the background sleep job.

## Mini task — find-hog.sh
```bash
#!/bin/bash
# find-hog.sh - prints the top 3 CPU-consuming processes
echo "Top 3 CPU-consuming processes:"
ps aux --sort=-%cpu | head -4
```
- Made executable with `chmod +x find-hog.sh`, ran with `./find-hog.sh`
- Output showed `snapfuse` and `snapd` processes as top CPU consumers on this sandbox — makes sense since those are the system's active background services in this environment.

## Notes / takeaways
- Signal numbers aren't interchangeable — `kill -9` (force kill) vs `kill -10` (SIGUSR1) behave completely differently. Worth memorizing the common ones: 1 (HUP), 9 (KILL), 15 (TERM, default).
- Priority changes that favor a process (lower nice value) need root — this is a security-sensible default, similar to least-privilege thinking from bug bounty work.
- Wrote and ran my first actual shell script today, not just copy-pasted commands.

## Status
✅ Day 4 complete — process management, signals, renice permissions, first shell script (find-hog.sh)

# Day 5 — systemd & Services

## Goal
Understand how systemd manages services, and create/run a custom service.

## Environment
Practiced on a local Ubuntu VM (not KillerCoda this time).

## Commands used
```bash
systemctl list-units --type=service
systemctl status cron
systemctl status ssh / sshd
sudo systemctl stop cron
sudo systemctl start cron
sudo systemctl restart cron
systemctl is-enabled cron
sudo systemctl disable cron
sudo systemctl enable cron
nano myservice.sh
chmod +x myservice.sh
sudo mv myservice.sh /usr/local/bin/myservice.sh
sudo nano /etc/systemd/system/myservice.service
sudo systemctl daemon-reload
sudo systemctl start myservice
systemctl status myservice
cat /tmp/myservice.log
sudo systemctl stop myservice
```

## What happened (real troubleshooting)

**ssh/sshd not found**
- `systemctl status ssh` and `systemctl status sshd` both returned "Unit could not be found."
- Reason: SSH server isn't installed by default on this system — the ssh *client* is present, but the service only exists once `openssh-server` is installed. Picked `cron` instead since it's active by default.

**Permission required to stop a service**
- `systemctl stop cron` (no sudo) → failed: "Access denied... requires interactive authentication."
- `sudo systemctl stop cron` → worked.
- Lesson: read-only actions like `status` and `list-units` don't need root, but state-changing actions (start/stop/restart/enable/disable) do.

**enable vs start — confirmed the distinction**
- `is-enabled cron` → `disabled` initially (won't auto-start on boot)
- `sudo systemctl enable cron` → created a symlink in `/etc/systemd/system/multi-user.target.wants/` pointing to the actual unit file — this symlink IS what "enabled" means under the hood
- Confirmed `disable` removes that same symlink
- This makes clear: **enable/disable controls boot-time behavior, start/stop controls current runtime state** — independent of each other

## Custom service — myservice.service

**Script:**
```bash
#!/bin/bash
while true; do
  echo "Service running at $(date)" >> /tmp/myservice.log
  sleep 10
done
```

**Unit file (`/etc/systemd/system/myservice.service`):**
```ini
[Unit]
Description=My Test Service

[Service]
ExecStart=/usr/local/bin/myservice.sh
Restart=always

[Install]
WantedBy=multi-user.target
```

**Result:**
- `systemctl status myservice` showed `active (running)`, Main PID tracked correctly, and the CGroup tree showed both the bash script AND its child `sleep 10` process — systemd tracks the whole process tree, not just the top one.
- `cat /tmp/myservice.log` confirmed new lines appearing every ~10 seconds, timestamps matching exactly.

## Notes / takeaways
- The enable/disable vs start/stop distinction finally clicked by actually toggling both independently and checking `is-enabled` each time — this is a common interview question and now I can explain it from having broken it myself.
- systemd tracks the full process tree of a service (parent script + child processes), which matters for things like `systemctl stop` correctly killing everything, not just the main PID.
- Real troubleshooting > tutorial happy-path: hitting the "ssh not found" and "permission denied" errors taught more than if everything had worked first try.

## Status
✅ Day 5 complete — systemd service lifecycle, enable/disable vs start/stop, wrote and ran a custom service

# Day 6 — Mini Project: Automated Server Setup Script

## Goal
Combine filesystem, permissions, process management, and systemd knowledge from Days 2-5 into one bootstrap script — and test how it behaves when run more than once.

## Environment
Practiced on a local Ubuntu VM.

## Script — setup.sh
```bash
#!/bin/bash
set -e

APP_USER="appuser"
APP_DIR="/opt/myapp"

echo "Creating user $APP_USER..."
sudo useradd -m -s /bin/bash "$APP_USER" || echo "User already exists"

echo "Creating app directory..."
sudo mkdir -p "$APP_DIR"
sudo chown "$APP_USER":"$APP_USER" "$APP_DIR"
sudo chmod 755 "$APP_DIR"

echo "Writing app script..."
sudo tee "$APP_DIR/run.sh" > /dev/null <<'EOF'
#!/bin/bash
while true; do
  echo "App running at $(date)" >> /opt/myapp/app.log
  sleep 10
done
EOF
sudo chmod +x "$APP_DIR/run.sh"
sudo chown "$APP_USER":"$APP_USER" "$APP_DIR/run.sh"

echo "Creating systemd service..."
sudo tee /etc/systemd/system/myapp.service > /dev/null <<EOF
[Unit]
Description=My App Service

[Service]
User=$APP_USER
ExecStart=$APP_DIR/run.sh
Restart=always

[Install]
WantedBy=multi-user.target
EOF

echo "Starting service..."
sudo systemctl daemon-reload
sudo systemctl enable myapp
sudo systemctl start myapp

echo "Done. Checking status:"
systemctl status myapp --no-pager
```

## Run 1 — first execution
- User created, directory created, script written, service created, enabled, and started
- `systemctl status myapp` → `active (running)`, Main PID 1001, child process `date` under CGroup
- `tail -f /opt/myapp/app.log` confirmed new lines appending every 10 seconds

## Run 2 — ran the exact same script again (idempotency test)

| Step | What happened |
|---|---|
| `useradd` | `useradd: user 'appuser' already exists` → caught by the `\|\| echo "User already exists"` fallback, script did NOT crash |
| `mkdir -p` | Silent success — `-p` flag means "create if missing, do nothing if it exists" — no error |
| `chown`/`chmod` | Ran again with no issue — reapplying the same ownership/permissions is harmless |
| Writing run.sh via `tee` | Overwrote the file with identical content — no error, no duplication |
| systemd service file | Overwritten with identical content via `tee` |
| `daemon-reload` + `enable` + `start` | No error — `systemctl start` on an already-running service is a no-op, it just confirms it's already active |
| Result | Service still shows `active (running)` — but notice the **start timestamp stayed the same** ("since Mon 2026-08-31 09:04:47 UTC; 1min 20s ago") — meaning the service was NOT restarted, just confirmed already running. CGroup PID also unchanged (1001), only the child process changed from `date` to `sleep 10` since it's mid-loop. |

## Why it didn't break — idempotency
This script is (mostly) **idempotent** — running it multiple times produces the same end state without errors, because:
- `useradd` failure is caught with `||`
- `mkdir -p` is naturally idempotent
- `tee` overwriting a file with the same content is harmless
- `systemctl enable`/`start` are no-ops if already enabled/running

**What's NOT truly idempotent here:** if I changed `run.sh` and reran the script, `tee` would update the file on disk, but the *already-running* process wouldn't pick up the change — I'd need `systemctl restart myapp` for that. The script currently doesn't restart the service to apply changes, only starts it if it's not running.

## Notes / takeaways
- This is my first real taste of "idempotency" — a core concept in DevOps tooling (Ansible, Terraform are built entirely around this idea: running the same config repeatedly should converge to the same state, not error out or duplicate work).
- `set -e` combined with `||` fallbacks is how you selectively allow expected failures (like "user already exists") while still failing hard on real errors.
- Running services as a dedicated non-root user (`User=$APP_USER` in the unit file) is a least-privilege practice — directly connects to security instincts from bug bounty work.
- Found a real gap: script doesn't restart the service on config changes — noting this as something to improve later (could add a restart step conditionally).

## Status
✅ Day 6 complete — Week 1 wrapped up. Built an idempotent server bootstrap script combining users, permissions, file scripting, and systemd services.

---
## 🎉 Week 1 Complete — Linux Fundamentals
Covered: filesystem hierarchy, permissions/ownership, process management, systemd services, and a combined mini project. Ready to move into Week 2: Bash Scripting + Networking.
