# WSL Ubuntu Recovery & Selective Migration Guide

> Personal recovery guide after reinstalling Windows.
>
> Goal:
> Recover only selected repositories and files from an old Ubuntu WSL installation while keeping a clean new Ubuntu environment.

---

# Current Architecture

## Active Ubuntu

Stored on:

```text
D:\WSL\Ubuntu\ext4.vhdx
```

Verify:

```powershell
wsl -l -v
```

Expected:

```text
Ubuntu
docker-desktop
```

Launch:

```powershell
wsl -d Ubuntu
```

Linux home:

```text
/home/rawad
```

---

# Recovery Ubuntu

Old Ubuntu backup:

```text
D:\old WSL\Ubuntu\ext4.vhdx
```

Size:

```text
~5-6 GB
```

This contains repositories such as:

```text
/home/rawad
├── node-readiness-controller
├── nrc
├── nrctest
└── pipecd
```

---

# Important Discovery

The following files were NOT the Ubuntu distro:

```text
F:\OLD D\Docker\wsl\main\ext4.vhdx
```

and

```text
F:\OLD D\Docker\wsl\disk\docker_data.vhdx
```

These contained Docker Desktop internal data.

Do not waste time trying to recover repositories from them.

---

# Recreate Ubuntu On D Drive

If Ubuntu gets installed on C: after a Windows reinstall:

## Export

```powershell
wsl --export Ubuntu D:\WSLBackup\ubuntu.tar
```

## Shutdown

```powershell
wsl --shutdown
```

## Remove Ubuntu Registration

```powershell
wsl --unregister Ubuntu
```

## Create Destination

```powershell
mkdir D:\WSL
mkdir D:\WSL\Ubuntu
```

## Import

```powershell
wsl --import Ubuntu D:\WSL\Ubuntu D:\WSLBackup\ubuntu.tar --version 2
```

Verify:

```powershell
wsl -l -v
```

Launch:

```powershell
wsl -d Ubuntu
```

Verify:

```bash
cd ~
pwd
```

Expected:

```text
/home/rawad
```

---

# Register Old Ubuntu For Recovery

If old Ubuntu exists as:

```text
D:\old WSL\Ubuntu\ext4.vhdx
```

Register it:

```powershell
wsl --import-in-place Ubuntu-Old "D:\old WSL\Ubuntu\ext4.vhdx"
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

Launch:

```powershell
wsl -d Ubuntu-Old
```

Verify:

```bash
cd ~
ls
```

Example:

```text
node-readiness-controller
nrc
nrctest
pipecd
```

---

# Selective Migration Workflow

## Open Two Terminals

### Terminal 1

Old Ubuntu:

```powershell
wsl -d Ubuntu-Old
```

### Terminal 2

New Ubuntu:

```powershell
wsl -d Ubuntu
```

---

# Migrate Multiple Repositories

Inside Ubuntu-Old:

```bash
cd ~

tar -czf /tmp/repos.tar.gz \
node-readiness-controller \
nrc \
nrctest \
pipecd
```

Copy to Windows:

```bash
cp /tmp/repos.tar.gz /mnt/c/Users/Rawad/
```

---

# Import Into New Ubuntu

Inside Ubuntu:

```bash
cp /mnt/c/Users/Rawad/repos.tar.gz ~/

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

# Migrate Single Repository

Example:

```text
nrc
```

Old Ubuntu:

```bash
tar -czf /tmp/nrc.tar.gz nrc

cp /tmp/nrc.tar.gz /mnt/c/Users/Rawad/
```

New Ubuntu:

```bash
cp /mnt/c/Users/Rawad/nrc.tar.gz ~/

tar -xzf nrc.tar.gz
```

---

# Verify Repository Health

Example:

```bash
cd ~/nrc

git status

git remote -v
```

Expected:

```text
On branch ...
nothing to commit
```

---

# Cleanup

Remove archive from Ubuntu:

```bash
rm ~/repos.tar.gz
```

Remove archive from Windows:

```powershell
del C:\Users\Rawad\repos.tar.gz
```

Remove temporary archive from Ubuntu-Old:

```bash
rm /tmp/repos.tar.gz
```

---

# Keep Ubuntu-Old Until Everything Is Verified

Do NOT immediately delete Ubuntu-Old.

Verify:

* repositories open correctly
* Git works
* npm works
* pnpm works
* Docker works
* Cursor works

Only then remove recovery distro.

---

# Remove Recovery Distro

When everything is confirmed:

```powershell
wsl --unregister Ubuntu-Old
```

This removes only the recovery registration.

It does not affect:

```text
Ubuntu
```

which remains the active development environment.

---

# Daily Workflow

Launch Ubuntu:

```powershell
wsl -d Ubuntu
```

Open repository:

```bash
cd ~/nrc
cursor .
```

Examples:

```bash
cd ~/node-readiness-controller
cursor .
```

```bash
cd ~/nrctest
cursor .
```

```bash
cd ~/pipecd
cursor .
```

---

# Final Rule

Always keep active development inside:

```text
/home/rawad
```

which lives physically in:

```text
D:\WSL\Ubuntu\ext4.vhdx
```

Avoid developing inside:

```text
/mnt/c
```

because that consumes C drive space and is slower for Linux tooling.
