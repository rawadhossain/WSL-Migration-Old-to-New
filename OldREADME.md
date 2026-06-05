# WSL Recovery & Selective Migration Guide

### Recover selected repositories/files from OLD WSL Ubuntu → NEW WSL Ubuntu (stored on D drive)

> Personal recovery and migration guide after moving from old laptop → new laptop.
>
> Goal:
> Keep a clean Ubuntu environment on the new laptop while selectively importing only needed projects from the old Ubuntu.

---

# Final Architecture

Current setup:

```text
NEW SSD

C:
├── Windows
└── Small system data

D:
├── WSL
│   └── Ubuntu
│       └── ext4.vhdx   ← ACTIVE Ubuntu filesystem
│
└── WSLBackup
```

Inside active Ubuntu:

```text
/home/rawad
├── nrc
├── nrctest
├── node-readiness-controller
└── pipecd
```

Old SSD:

```text
F:
└── Users
    └── Rawad
        └── AppData
            └── Local
                └── wsl
                    └── {GUID}
                        └── ext4.vhdx
```

Imported as:

```text
Ubuntu-Old
```

---

# Important Concepts

Current active environment:

```bash
wsl -d Ubuntu
```

Source/archive environment:

```bash
wsl -d Ubuntu-Old
```

Never work permanently inside:

```text
Ubuntu-Old
```

Use it only to recover files.

---

# Verify Current Setup

Open PowerShell:

```powershell
wsl -l -v
```

Expected:

```text
NAME              STATE
Ubuntu            Running
Ubuntu-Old        Stopped
docker-desktop    Running
```

---

# Storage Rules

Inside Ubuntu:

```bash
~/repo
```

Stored physically in:

```text
D:\WSL\Ubuntu\ext4.vhdx
```

Consumes:

```text
D drive
```

Examples:

```bash
git clone
npm install
pnpm install
cursor .
```

All grow D drive.

---

Inside:

```bash
/mnt/c
```

Consumes:

```text
C drive
```

Avoid development there.

---

# Daily Workflow

Open terminal:

```bash
wsl
```

Open project:

```bash
cd ~/nrc
cursor .
```

Examples:

```bash
cd ~/nrctest
cursor .
```

```bash
cd ~/node-readiness-controller
cursor .
```

---

# Selective Migration Workflow

Use when you want:

```text
folder
repo
file
workspace
config
```

from OLD Ubuntu.

Examples:

```text
metaflow
builder
research
node-readiness-controller
```

---

# METHOD A — Move ONE Folder / Repo

Example:

```text
metaflow
```

---

## STEP 1 — Open Windows Terminal

Open:

```text
Windows Terminal
```

Open LEFT tab.

---

## STEP 2 — Open OLD Ubuntu

Run:

```bash
wsl -d Ubuntu-Old
```

Go home:

```bash
cd ~
pwd
ls
```

Expected:

```text
/home/rawad
```

---

## STEP 3 — Create Archive

Replace folder name.

Run:

```bash
tar -czf /tmp/metaflow.tar.gz metaflow
```

Wait.

Finished when prompt returns.

Verify:

```bash
ls -lh /tmp/metaflow.tar.gz
```

---

## STEP 4 — Move Archive To Windows

Run:

```bash
cp /tmp/metaflow.tar.gz /mnt/c/Users/Rawad/
```

Verify:

```bash
ls -lh /mnt/c/Users/Rawad/metaflow.tar.gz
```

---

## STEP 5 — Open SECOND Terminal Tab

Press:

```text
+
```

Run:

```bash
wsl -d Ubuntu
```

---

## STEP 6 — Go Home

Run:

```bash
cd ~
pwd
```

Expected:

```text
/home/rawad
```

---

## STEP 7 — Copy Archive Into NEW Ubuntu

Run:

```bash
cp /mnt/c/Users/Rawad/metaflow.tar.gz ~/
```

---

## STEP 8 — Extract

Run:

```bash
tar -xzf metaflow.tar.gz
```

Verify:

```bash
ls
```

Expected:

```text
metaflow
```

---

## STEP 9 — Cleanup

NEW Ubuntu:

```bash
rm ~/metaflow.tar.gz
```

OLD Ubuntu:

```bash
rm /tmp/metaflow.tar.gz
```

Optional:

```bash
rm /mnt/c/Users/Rawad/metaflow.tar.gz
```

Done.

---

# METHOD B — Move MULTIPLE Folders

Example:

```text
folder1
folder2
folder3
```

---

## LEFT TAB → OLD Ubuntu

Run:

```bash
wsl -d Ubuntu-Old
```

Create archive:

```bash
tar -czf /tmp/repos.tar.gz \
folder1 \
folder2 \
folder3
```

Verify:

```bash
ls -lh /tmp/repos.tar.gz
```

Copy:

```bash
cp /tmp/repos.tar.gz /mnt/c/Users/Rawad/
```

---

## RIGHT TAB → NEW Ubuntu

Run:

```bash
wsl -d Ubuntu
```

Copy:

```bash
cp /mnt/c/Users/Rawad/repos.tar.gz ~/
```

Extract:

```bash
tar -xzf repos.tar.gz
```

Verify:

```bash
ls
```

Cleanup:

```bash
rm ~/repos.tar.gz
rm /mnt/c/Users/Rawad/repos.tar.gz
```

Done.

---

# Move ONLY Specific Files

Example:

```text
README.md
.env
notes.txt
```

OLD Ubuntu:

```bash
tar -czf /tmp/files.tar.gz \
README.md \
.env \
notes.txt
```

Copy:

```bash
cp /tmp/files.tar.gz /mnt/c/Users/Rawad/
```

NEW Ubuntu:

```bash
cp /mnt/c/Users/Rawad/files.tar.gz ~/
tar -xzf files.tar.gz
```

---

# Verify Repository Health

Open:

```bash
cd ~/nrc
cursor .
```

Inside terminal:

```bash
git status
```

Expected:

```text
On branch ...
nothing to commit
```

Verify remote:

```bash
git remote -v
```

---

# Check WSL Uses D Drive

PowerShell:

```powershell
dir D:\WSL\Ubuntu
```

Expected:

```text
ext4.vhdx
```

Monitor size growth.

---

# Check Disk Usage

Inside Ubuntu:

```bash
df -h ~
```

See Linux filesystem usage.

---

# Cleanup Old Ubuntu (Optional)

Only after confirming:

- all repos recovered
- SSH works
- Cursor works
- no missing files

Remove:

```powershell
wsl --unregister Ubuntu-Old
```
