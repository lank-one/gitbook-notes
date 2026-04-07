---
icon: list-check
---

# CheatSheet

## Introduction

In this section, I will share a list of the commands that I found most useful in the different phases of the exam, with the options that, in my case, worked best and allowed me to pass.

## Recon

#### Fping

```
fping -a -g
```

-a: This option indicates that the IP addresses of hosts that are active on the network should be displayed.\
-g: This argument specifies that the ping is to an IP range.

#### Nmap

```
nmap -sn 192.168.100.0/24
```

-sn: This option indicates a “ping scan”; it does not scan ports, it only checks if the hosts are active.

#### [Ping Sweep](https://www.rubyguides.com/2012/02/cli-ninja-ping-sweep/) :link:

Another alternative to network reconnaissance, which would be the same as the previous one from Nmap. Copy and paste directly into the terminal.

Linux

```
for i in {1..254} ;do (ping -c 1 192.168.1.$i | grep “bytes from” &) ;done
```

Windows

```
for /L %i in (1,1,255) do @ping -n 1 -w 200 192.168.1.%i > nul && echo 192.168.1.%i is up.
```

## Enumeration

Here I will group together the commands for identifying ports, services, versions, users, etc.

### Nmap

The nmap tool will be our best ally for performing a thorough reconnaissance of the hosts on the network and the services they are running.

```
nmap -sC -sV -O 
```

-sC: Execution of default scripts, which allow us to identify vulnerabilities to which the service may be vulnerable or obtain additional information.

-sV: Identifies the version of the service running on the port, allowing us to know if that service is vulnerable to any known vulnerabilities.

-O: Determines the operating system running on the target system.

### Curl

During my exam, I encountered two machines running CMS, each running a different one. To obtain the information requested in one of the questions, I used the following commands:

```
curl | grep ‘CMS name’
```

```
curl -s <IP or domain + configuration file path>
```

### Metasploit

Metasploit has enumeration modules for different services, which is a good alternative if you don't know specific tools or methods.

```
msfconsole -q
msf6 > search smb_enumusers
msf6 > use 1
msf6 auxiliary(scanner/smb/smb_enumusers) > set RHOSTS msf6 auxiliary(scanner/smb/smb_enumusers) 
msf6 auxiliary(scanner/smb/smb_enumusers) > set SMBUser msf6 auxiliary(scanner/smb/smb_enumusers)
msf6 auxiliary(scanner/smb/smb_enumusers) > set SMBPass msf6 auxiliary(scanner/smb/smb_enumusers)
msf6 auxiliary(scanner/smb/smb_enumusers) > run
```

### Enum4linux

For a Linux system, one of the enumeration tools you can use is:

```
enum4linux <IP>
```

## Exploitation

### Metasploit

Thanks to its modules, you can use it to exploit multiple services.

Basic commands:

```
// Start console without banner
msfconsole -q

// Search for a module
msf6 > search

// Use module
msf6 > use

// Set global option
msf6 > setg

// Set target host
msf6 > set RHOSTS

// Set target port
msf6 > set RPORT

// Configured module options
msf6 > options

// Module information
msf6 > info

// Run module
msf6 > run
```

### Brute Force

In the course labs, brute force is a recurring technique, and this is also the case in the exam.

It will make it much easier for you to obtain user credentials. For me, the winning combination was hydra with rockyou.txt.

```
hydra -l <user> -P /usr/share/wordlists/rockyou.txt <service>://<IP>
```

## Post-Exploitation

Once you have gained access to the target system, you will need to navigate through it to find what you are looking for. To do this, you will need to use system commands, depending on the system you are attacking.\
Here are some examples of commands I had to use:

```
// Identify which user you are
whoami

// List system users
net user

// Download a file via FTP
ftp> get file.txt

// Upload a file via FTP
ftp> put file.txt

// Switch from Meterpreter to a shell
meterpreter> shell
/bin/bash -i
```
