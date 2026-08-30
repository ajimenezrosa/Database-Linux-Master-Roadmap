# 📘 Complete Linux Training Course to Get Your Dream IT Job

## 📚 Syllabus — Module 3: System Access and File System

---

# Module 3 – System Access and File System

> **"Before you can administer Linux, you must first know how to access it, navigate it, and understand how it organizes every file on the system."**

---

# 🎯 Learning Objectives

By the end of this module, you will be able to:

* Access a Linux machine locally and remotely.
* Install and configure PuTTY.
* Understand Linux networking basics.
* Navigate the Linux filesystem efficiently.
* Create and manage files and directories.
* Understand Linux file types.
* Search for files using `find` and `locate`.
* Understand symbolic and hard links.
* Use wildcards to manipulate multiple files.
* Change user passwords securely.

---

# 🖥️ Accessing a Linux System

Before administering Linux, you must first gain access to the operating system.

There are two primary ways to access a Linux machine:

## Local Access

You are physically sitting in front of the computer.

Examples:

* Desktop computer
* Laptop
* Virtual Machine

---

## Remote Access

You connect to another Linux server over a network.

Examples:

* SSH
* PuTTY
* OpenSSH
* SecureCRT
* MobaXterm

Remote administration is the standard method used by Linux System Administrators and Cloud Engineers.

---

# 🌐 Remote Administration with SSH

SSH (Secure Shell) is the industry standard protocol for securely accessing Linux systems remotely.

Benefits:

* Encrypted communication
* Secure authentication
* File transfers
* Remote command execution
* Port forwarding

Default SSH Port:

```text
22
```

---

# 💻 Downloading and Installing PuTTY

PuTTY is one of the most popular SSH clients for Windows.

It allows administrators to remotely manage Linux servers through a secure encrypted connection.

Typical workflow:

1. Install PuTTY.
2. Enter the Linux server IP.
3. Select SSH.
4. Connect.
5. Authenticate.
6. Start working.

---

# 🌍 Understanding Network Commands

Every Linux administrator must know how to identify network configuration.

## Legacy Command

```bash
ifconfig
```

Historically used to display network interfaces.

Today it is considered legacy and is no longer installed by default on many Linux distributions.

---

## Modern Command

```bash
ip addr
```

or

```bash
ip a
```

Displays:

* IP Address
* MAC Address
* Interface State
* Broadcast Address
* Network Configuration

---

Other useful commands:

```bash
ip route
```

```bash
hostname
```

```bash
hostname -I
```

```bash
ping google.com
```

---

# 🔌 Connecting to Your Linux Virtual Machine Using PuTTY

Before connecting:

✅ Linux VM must be running.

✅ Network Adapter configured.

✅ SSH Service installed.

Check SSH service:

```bash
systemctl status sshd
```

Enable SSH:

```bash
sudo systemctl enable sshd
```

Start SSH:

```bash
sudo systemctl start sshd
```

Connect:

```text
Host Name:
192.168.1.100
```

Port:

```text
22
```

Protocol:

```text
SSH
```

---

# ⚠️ Important Things to Remember in Linux

Linux behaves differently from Windows.

Keep these rules in mind:

✔ Linux is case-sensitive.

```text
File.txt
```

is NOT the same as

```text
file.txt
```

---

Everything in Linux is treated as a file.

Examples:

* Hard drives
* USB devices
* Network sockets
* Printers
* Directories

---

Always verify commands before pressing Enter.

A single incorrect command executed as root may damage the operating system.

---

# 📁 Introduction to the Linux File System

Linux stores everything inside a single hierarchical directory structure.

Unlike Windows, Linux does not use drive letters such as:

```text
C:
D:
E:
```

Everything begins at one directory:

```text
/
```

Known as:

**Root Directory**

---

# 🌳 Linux File System Structure

The root directory contains the most important system directories.

| Directory | Purpose                 |
| --------- | ----------------------- |
| /         | Root Directory          |
| /bin      | Essential user commands |
| /boot     | Boot Loader             |
| /dev      | Devices                 |
| /etc      | Configuration Files     |
| /home     | User Home Directories   |
| /lib      | Shared Libraries        |
| /media    | Removable Media         |
| /mnt      | Temporary Mount Points  |
| /opt      | Optional Software       |
| /proc     | Kernel Information      |
| /root     | Home of Root User       |
| /run      | Runtime Data            |
| /srv      | Services                |
| /sys      | Kernel Devices          |
| /tmp      | Temporary Files         |
| /usr      | Applications            |
| /var      | Variable Data and Logs  |

---

# 🧭 File System Navigation Commands

Most frequently used commands:

Current directory

```bash
pwd
```

List files

```bash
ls
```

Detailed list

```bash
ls -l
```

Hidden files

```bash
ls -la
```

Change directory

```bash
cd
```

Return home

```bash
cd
```

Go back

```bash
cd ..
```

Root directory

```bash
cd /
```

---

# 📍 Linux File System Paths

Two types of paths exist.

## Absolute Path

Starts from:

```text
/
```

Example

```text
/home/alex/Documents/report.txt
```

---

## Relative Path

Starts from your current location.

Example

```text
Documents/report.txt
```

---

# 📄 Directory Listing Overview

Useful commands:

```bash
ls
```

```bash
ls -lh
```

```bash
ls -ltr
```

```bash
tree
```

Example output:

```text
Documents
Downloads
Desktop
Pictures
Videos
```

---

# 📂 Creating Files and Directories

Create directory

```bash
mkdir LinuxLab
```

Nested directories

```bash
mkdir -p Labs/Module3/Practice
```

Create file

```bash
touch notes.txt
```

Multiple files

```bash
touch file1 file2 file3
```

---

# 📦 Linux File Types

Linux supports many file types.

| Type | Description      |
| ---- | ---------------- |
| -    | Regular File     |
| d    | Directory        |
| l    | Symbolic Link    |
| c    | Character Device |
| b    | Block Device     |
| s    | Socket           |
| p    | Named Pipe       |

Example:

```bash
ls -l
```

The first character indicates the file type.

---

# 🔍 Finding Files and Directories

## Using find

Example:

```bash
find /home -name "*.txt"
```

Find directories

```bash
find / -type d
```

Find files larger than 100 MB

```bash
find / -size +100M
```

---

## Using locate

```bash
locate passwd
```

If locate database is outdated:

```bash
sudo updatedb
```

---

# ⚖️ Difference Between find and locate

| find                     | locate                   |
| ------------------------ | ------------------------ |
| Searches live filesystem | Uses database            |
| Slower                   | Faster                   |
| Always current           | Database may be outdated |
| Very flexible            | Very fast                |

General rule:

* Use **find** when accuracy matters.
* Use **locate** when speed matters.

---

# 🔐 Changing Password

Change your own password:

```bash
passwd
```

Root changes another user's password:

```bash
sudo passwd username
```

Good passwords should include:

* Uppercase letters
* Lowercase letters
* Numbers
* Special characters
* Minimum 12 characters

---

# ⭐ Wildcards

Wildcards simplify file operations.

## *

Matches everything.

```bash
ls *.txt
```

---

## ?

Matches one character.

```bash
ls file?.txt
```

---

## $

Represents variables.

```bash
echo $HOME
```

```bash
echo $PATH
```

---

## ^

Frequently used in regular expressions.

Example:

```bash
grep "^root" /etc/passwd
```

Meaning:

Starts with "root".

---

# 🔗 Soft Links and Hard Links

Linux allows multiple references to the same file.

## Symbolic Link

Shortcut to another file.

Create:

```bash
ln -s original.txt shortcut.txt
```

---

## Hard Link

Points directly to the file inode.

Create:

```bash
ln original.txt hardlink.txt
```

---

Comparison

| Symbolic Link              | Hard Link         |
| -------------------------- | ----------------- |
| Can cross filesystems      | Cannot            |
| Can link directories       | Usually No        |
| Breaks if original removed | Continues working |

---

# 🖼️ Opening Images Through the GUI

Most Linux desktop environments allow images to be opened by simply double-clicking them.

From the terminal you can also use:

```bash
xdg-open image.jpg
```

Or on GNOME:

```bash
eog image.jpg
```

Supported image formats include:

* JPG
* PNG
* GIF
* BMP
* TIFF
* WEBP

---

# 📝 Module Quiz

1. What is SSH?
2. What command replaces `ifconfig`?
3. What is the Linux Root Directory?
4. What is the difference between an absolute path and a relative path?
5. What command creates directories?
6. What is the difference between `find` and `locate`?
7. What is a symbolic link?
8. What is a hard link?
9. Which wildcard matches every file?
10. Which command changes a user's password?

---

# 📚 Homework

### Practice 1

Install and configure SSH on your Linux virtual machine.

---

### Practice 2

Connect remotely using PuTTY.

---

### Practice 3

Create the following directory structure:

```text
LinuxLab
├── Documents
├── Scripts
├── Pictures
└── Backups
```

---

### Practice 4

Create ten empty files using `touch`.

---

### Practice 5

Use `find` to locate all `.conf` files inside `/etc`.

---

### Practice 6

Create one symbolic link and one hard link.

---

### Practice 7

Change your user password.

---

### Practice 8

Navigate the filesystem using only terminal commands without using the graphical interface.

---

# 📎 Handouts

* Linux Filesystem Hierarchy (FHS) Reference
* Linux Navigation Commands Cheat Sheet
* SSH Quick Reference Guide
* PuTTY Configuration Guide
* Linux Wildcards Reference
* File Types Quick Guide
* `find` vs `locate` Comparison Sheet
* Symbolic Links vs Hard Links Reference Card

---

# 🎓 Module Summary

In this module, you learned how Linux systems are accessed locally and remotely, configured SSH connectivity, and explored the Linux filesystem hierarchy. You practiced navigating directories, creating files, identifying file types, searching for information efficiently, working with links, and using wildcards to simplify administration tasks.

These are foundational skills used daily by Linux System Administrators, DevOps Engineers, Cloud Engineers, Database Administrators (DBAs), Site Reliability Engineers (SREs), and Cybersecurity Professionals. Mastering them will significantly improve your confidence and efficiency when working in any Linux environment.

---
