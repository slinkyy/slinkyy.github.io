---
layout: default
title: Nmap
parent: Enumeration
has_children: false
nav_order: 6
---

# Nmap
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

### Parallel Nmap Scanning

Executing an aggressive nmap scan against multiple targets i.e a subnet of hosts can be done using the following script:

`https://github.com/slinkyy/tools/blob/main/parallel_nmap.sh`

Firstly obtain the host addresses to be scanned using nmaps ping sweep against the subnet

```
nmap -sn <TARGET NETWORK>/<MASK>
```

*Note: Change the `--jobs` setting based on your needs*

```
#!/bin/bash
[ "$UID" -eq 0 ] || exec sudo bash "$0" "$@"

mkdir -p scans
echo "[*] Scan directory created"
echo "[*] Creating live_host output directories"
cat live_hosts.txt | while read ip; do mkdir -p scans/${ip}; done
echo "[*] Creating parallel script"
cat live_hosts.txt | while read ip; do echo "sudo nmap -A -sV -Pn -p- --script=default,vuln --open -oA scans/${ip}/${ip} $ip"; done > scan_all_ips.out
echo "[*] Executing parallel script"
parallel --jobs 32 < scan_all_ips.out
echo "[*] Changing ownership of output files"
sudo chown <CHANGEME>:<CHANGEME> -R scans
```