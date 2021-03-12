---
layout: default
title: Linux
parent: Enumeration
has_children: false
nav_order: 1
---

# Linux
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Automated Enumeration

### linPEAS

### linuxprivchecker

### LinEnum

## Manual Enumeration

**Enumerating Users**

On Linux-based systems, we can use the id command to gather user context information, The output reveals the user were operating as, which has a User Identifier (UID) and Group Identifier (GID)

```
id
```

To enumerate users on a Linux-based system, we can simply read the contents of the `/etc/passwd` file.


```
cat /etc/passwd
```

### Operating System Version and Architecture

We can discover the hostname with the aptly-named hostname command, which is installed on both Windows and Linux.

A machine’s hostname can often provide clues about its functional roles. More often than not, the hostnames will include identifiable abbreviations such as web for a web server, db for a database server, dc for a domain controller, etc.

```
hostname
```

The /etc/issue and /etc/*-release files contain specific operating system and architecture information. We can also issue the `uname -a` command.

```
cat /etc/issue
cat /etc/*-release
uname -a
```

### Running Processes and Services

On Linux, we can list system processes (including those run by privileged users) with the `ps` command. We’ll use the `a` and `x` flags to list all processes with or without a tty and the `u` flag to list the processes in a user-readable form.

```
ps aux
```


### Networking Information

we can list the TCP/IP configuration of every network adapter with either `ifconfig` or `ip`. Both commands accept the a flag to display all information available.
 
```
ip a
ifconfig -a
``` 

We can display network routing tables with either `route` or `routel`, depending on the Linux flavor and version. 

```
/sbin/route
/usr/bin/routel
```

we can list all connections with `-a`, avoid hostname resolution (which may stall the command execution) with -n, and list the process name the connection belongs to with `-p`.

```
ss -anp
netstat -antup
```

### Firewall Status and Rules

On Linux-based systems, we must have root privileges to list firewall rules with iptables.However, depending on how the firewall is configured, we may be able to glean information about the rules as a standard user. 

For example, the iptables-persistent package on Debian Linux saves firewall rules in specific files under the `/etc/iptables` directory by default. These files are used by the system to restore netfilter rules at boot time. These files are often left with weak permissions, allowing them to be read by any local user on the target system. We can also search for files created by the iptables-save command, which is used to dump the firewall configuration to a file specified by the user. This file is then usually used as input for the `iptables-restore` command and used to restore the firewall rules at boot time. 

If a system administrator had ever run this command, we could search the configuration directory `/etc` or grep the file system for iptables commands to locate the file. If the file has insecure permissions, we could use the contents to infer the firewall configuration rules running on the system. 

### Scheduled Tasks

The Linux-based job scheduler is known as Cron. Scheduled tasks are listed under the `/etc/cron.*` directories, where `*` represents the frequency the task will run on. For example, tasks that will be run daily can be found under /etc/cron.daily. 

Each script is listed in its own subdirectory. 

```
cat /etc/crontab
cat /etc/cron.daily
cat /etc/cron.monthly
cat /etc/cron.weekly
```

### Installed Applications and Patch Levels

Linux-based systems use a variety of package managers. For example, Debian-based Linux distributions use `dpkg` while Red Hat based systems use `rpm`.

```
dpkg -l
```

### Readable/Writable Files and Directories
 
Searching for every directory writable by the current user on the target system. We search the whole root directory (/) and use the `-writable` argument to specify the attribute we are interested in. We also use `-type d` to locate directories, and we filter errors with `2>/dev/null`

```
find / -writable -type d 2>/dev/null
```

### Unmounted Disks

On Linux-based systems, we can use the mount command to list all mounted filesystems. In addition, the `/etc/fstab` file lists all drives that will be mounted at boot time.

```
cat /etc/fstab
mount
```

Furthermore, we can use lsblk to view all available disks.

```
/bin/lsblk
``` 

### Kernel Versions for Exploits

Identify the OS and version of the target machine to find a kernel exploit for
 
```
cat /etc/issue
```

Inspect the kernel version

```
uname -r 
```
 
Inspect system architecture

```
arch
```

### Device Drivers and Kernel Modules

On Linux, we can enumerate the loaded kernel modules using lsmod without any additional arguments. 

```
lsmod
```

Once we have the list of loaded modules and identify those we want more information about, like libata in the above example, we can use modinfo to find out more about the specific module. Note that this tool requires a full pathname to run.

```
/sbin/modinfo libata
``` 

### Binaries That AutoElevate (SUID)

We can use the find command to search for SUID-marked binaries. In this case, we are starting our search at the root directory (/), looking for files `-type f` with the SUID bit set, `-perm -u=s` and discarding all error messages `2>/dev/null`
 
```
find / -perm -u=s -type f 2>/dev/null
```

If there are SUID binaries on the box, it is worth checking `https://gtfobins.github.io/gtfobins`
to see if its possible to elevate.



