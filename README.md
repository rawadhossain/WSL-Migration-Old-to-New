# WSL Recovery & Selective Migration Playbook

## Purpose

This guide is used after:

- Fresh Windows installation
- New laptop setup
- WSL reinstall
- Recovery from old Ubuntu VHDX

Goal:

Recover selected repositories from an old Ubuntu WSL installation into a fresh Ubuntu installation while keeping the active Ubuntu filesystem on D drive.

---

# Final Desired Architecture

```text
D:
├── WSL
│   └── Ubuntu
│       └── ext4.vhdx      ← ACTIVE Ubuntu
│
└── WSL-OLD
    └── Ubuntu
        └── ext4.vhdx      ← RECOVERY SOURCE
```

After migration:

```text
D:
└── WSL
    └── Ubuntu
        └── ext4.vhdx
```

---

# Phase 1 — Verify Current WSL State

Open PowerShell:

```powershell
wsl -l -v
```

Example:

```text
NAME              STATE           VERSION
docker-desktop    Running         2
```

This means Ubuntu is not installed yet.

---

# Phase 2 — Mount Old Ubuntu

Assume old Ubuntu filesystem exists at:

```text
D:\WSL-OLD\Ubuntu\ext4.vhdx
```

Register it:

```powershell
wsl --import-in-place Ubuntu-Old "D:\WSL-OLD\Ubuntu\ext4.vhdx"
```

Verify:

```powershell
wsl -l -v
```

Expected:

```text
NAME
Ubuntu-Old
docker-desktop
```

Open old Ubuntu:

```powershell
wsl -d Ubuntu-Old
```

Verify:

```bash
pwd
ls /home/rawad
```

Expected:

```text
node-readiness-controller
nrc
nrctest
pipecd
```

If repositories appear, recovery source is confirmed.

---

# Phase 3 — Install Fresh Ubuntu

Install Ubuntu:

```powershell
wsl --install -d Ubuntu
```

Create Linux user:

```text
rawad
```

Verify:

```powershell
wsl -l -v
```

Expected:

```text
Ubuntu
Ubuntu-Old
docker-desktop
```

---

# Phase 4 — Move Ubuntu To D Drive

By default Ubuntu installs on C drive.

Export:

```powershell
wsl --export Ubuntu D:\ubuntu.tar
```

Shutdown WSL:

```powershell
wsl --shutdown
```

Remove Ubuntu from C drive:

```powershell
wsl --unregister Ubuntu
```

Create destination:

```powershell
mkdir D:\WSL\Ubuntu
```

Import Ubuntu to D drive:

```powershell
wsl --import Ubuntu D:\WSL\Ubuntu D:\ubuntu.tar --version 2
```

Delete temporary archive:

```powershell
del D:\ubuntu.tar
```

Verify:

```powershell
Get-ChildItem D:\WSL\Ubuntu
```

Expected:

```text
ext4.vhdx
```

---

# Phase 5 — Verify Both Distros

PowerShell:

```powershell
wsl -l -v
```

Expected:

```text
Ubuntu
Ubuntu-Old
docker-desktop
```

---

# Phase 6 — Open Two Terminals

## Terminal 1 (Old Ubuntu)

Open:

```powershell
wsl -d Ubuntu-Old
```

Verify:

```bash
pwd
```

Expected:

```text
/home/rawad
```

Go home:

```bash
cd /home/rawad
```

---

## Terminal 2 (New Ubuntu)

Open:

```powershell
wsl -d Ubuntu
```

Verify:

```bash
pwd
```

Expected:

```text
/home/rawad
```

Keep both terminals open.

---

# Phase 7 — Create Archive In Old Ubuntu

## Terminal 1 (Ubuntu-Old)

Create archive:

```bash
tar -czf /tmp/repos.tar.gz \
nrc \
nrctest \
node-readiness-controller \
pipecd
```

Verify:

```bash
ls -lh /tmp/repos.tar.gz
```

Expected:

```text
repos.tar.gz
```

Copy archive to D drive:

```bash
cp /tmp/repos.tar.gz /mnt/d/
```

Verify:

```bash
ls -lh /mnt/d/repos.tar.gz
```

---

# Phase 8 — Import Into New Ubuntu

## Terminal 2 (Ubuntu)

Copy archive:

```bash
cp /mnt/d/repos.tar.gz ~/
```

Verify:

```bash
ls -lh ~/repos.tar.gz
```

Extract:

```bash
cd ~
tar -xzf repos.tar.gz
```

Verify:

```bash
ls
```

Expected:

```text
node-readiness-controller
nrc
nrctest
pipecd
```

---

# Phase 9 — Fix Ownership

Inside NEW Ubuntu:

```bash
sudo chown -R rawad:rawad \
~/nrc \
~/nrctest \
~/node-readiness-controller \
~/pipecd
```

---

# Phase 10 — Verify Repositories

Inside NEW Ubuntu:

```bash
cd ~/nrc
git status
```

```bash
cd ~/nrctest
git status
```

```bash
cd ~/node-readiness-controller
git status
```

```bash
cd ~/pipecd
git status
```

Expected:

```text
On branch ...
nothing to commit, working tree clean
```

---

# Phase 11 — Verify Cursor

```bash
cd ~/nrc
cursor .
```

```bash
cd ~/node-readiness-controller
cursor .
```

Verify projects open correctly.

---

# Phase 12 — Cleanup Temporary Files

## Terminal 2 (Ubuntu)

Remove archive:

```bash
rm ~/repos.tar.gz
```

---

## Terminal 1 (Ubuntu-Old)

Remove archive:

```bash
rm /tmp/repos.tar.gz
```

---

Remove Windows copy:

```bash
rm /mnt/d/repos.tar.gz
```

---

# Phase 13 — Keep Recovery Ubuntu

Keep Ubuntu-Old for several days.

Use:

```powershell
wsl -d Ubuntu
```

for daily work.

Verify:

- Git works
- Cursor works
- SSH works
- Docker works
- No missing repositories

---

# Phase 14 — Remove Recovery Ubuntu

After confirming everything:

Shutdown:

```powershell
wsl --shutdown
```

Remove recovery distro:

```powershell
wsl --unregister Ubuntu-Old
```

Optional:

Delete recovery files:

```powershell
Remove-Item D:\WSL-OLD -Recurse
```

---

# Final Daily Workflow

Open Ubuntu:

```powershell
wsl -d Ubuntu
```

Open project:

```bash
cd ~/nrc
cursor .
```

Other repositories:

```bash
cd ~/nrctest
cursor .
```

```bash
cd ~/node-readiness-controller
cursor .
```

```bash
cd ~/pipecd
cursor .
```

---

# Backup Rule

Before any future Windows reinstall:

```powershell
wsl --export Ubuntu D:\Backups\ubuntu-backup.tar
```

Store:

```text
ubuntu-backup.tar
```

on another drive.

This single file can restore the entire Ubuntu environment later.
