This is the introduction to the Cybersecurity homelab series where I create a 7 VM network with both Windows and Ubuntu to practice attack and defense scenarios. We will also use a suite of defense security tools based on Debian called Security Onion and of course, the attack suite of Kali Linux.

We will use Wazuh as our SIEM tool.

Here's a brief of each of our machines involved in the homelab.
### The Domain Controller - Windows Server 2025
| VM Name        | OS                  | Specs            | Storage | IPv4     | Account       | Password   |
| -------------- | ------------------- | ---------------- | ------- | -------- | ------------- | ---------- |
| pilgrimcorp-dc | Windows Server 2025 | 2 CPUs / 4096 MB | 50 Gb   | 10.0.0.5 | Administrator | @Deebodah1 |
This Windows Server will act as our central system to which everything is connected. This machine will have Active Directory installed and all other machines will connect to this using AD. It will also act as our intermediary DNS server which will allow all other devices to connect to the internet via this server.
### Windows Client - Windows 11 Enterprise
| VM Name                | OS                    | Specs            | Storage | IPv4       | Account                    | Password      |
| ---------------------- | --------------------- | ---------------- | ------- | ---------- | -------------------------- | ------------- |
| pilgrimcorp-win-client | Windows 11 Enterprise | 2 CPUs / 4096 MB | 80 Gb   | 10.0.0.100 | johnd@corp.pilgrimcorp.com | @password123! |
This is a standard workstation in an enterprise environment. This workstation is used by a user called `John Doe` 
### Linux Client - Ubuntu Desktop 22.04
| VM Name                  | OS                   | Specs           | Storage | IPv4       | Account            | Password      |
| ------------------------ | -------------------- | --------------- | ------- | ---------- | ------------------ | ------------- |
| pilgrimcorp-linux-client | Ubuntu Desktop 22.04 | 1 CPU / 2048 MB | 80 Gb   | 10.0.0.101 | janed@linux-client | @password123! |
This is the Ubuntu linux client. Why Linux in an Active Directory environment? Well, we can imagine this machine belongs to a developer who prefers using Linux instead of Windows for streamlined coding workflows. 
### Email Server - Ubuntu Server 22.04
| VM Name               | OS                  | Specs           | Storage | IPv4     | Account       | Password   |
| --------------------- | ------------------- | --------------- | ------- | -------- | ------------- | ---------- |
| pilgrimcorp-email-svr | Ubuntu Server 22.04 | 1 CPU / 2048 MB | 25 Gb   | 10.0.0.8 | Administrator | @Deebodah1 |
Ideally, even in enterprise environments, email is typically outsourced as a SaaS. Still, in order to get a greater in depth understanding of how email works, we will provision this server. This will also highlight the various attack vectors from the email side of security.
### The Defense - Security Onion
| VM Name             | OS             | Specs           | Storage | IPv4       | Account             | Password      |
| ------------------- | -------------- | --------------- | ------- | ---------- | ------------------- | ------------- |
| pilgrimcorp-sec-def | Security Onion | 1 CPU / 2048 MB | 55 Gb   | 10.0.0.103 | pilgrimcorp-sec-def | @password123! |
Security Onion is a distro based on Debian which comes with a whole bunch of pre-installed tools to setup our defenses.
### The Monitoring Station - Ubuntu Desktop 22.04
| VM Name             | OS                   | Specs            | Storage       | IPv4      | Account          | Password      |
| ------------------- | -------------------- | ---------------- | ------------- | --------- | ---------------- | ------------- |
| pilgrimcorp-sec-box | Ubuntu Desktop 22.04 | 2 CPUs / 4096 MB | 80 Gb* [Imp!] | 10.0.0.10 | sec-user@sec-box | @password123! |
This is our monitoring station. This is where our SIEM tool Wazuh will be installed. All the Wazuh agents will send logs to this machine. It is our own tiny SOC.
### The Attacker - Kali Linux
| VM Name  | OS                | Specs           | Storage | IPv4    | Account           | Password |
| -------- | ----------------- | --------------- | ------- | ------- | ----------------- | -------- |
| attacker | Kali Linux Latest | 1 CPU / 2048 MB | 25 Gb   | dynamic | attacker@attacker | attacker |
The attacker environment, comes with a host of pentesting and ethical hacking tools to help us simulate an attack on our network.

> [!warning] Simple Passwords
> The passwords have been purposefully kept simple to make a case in point about security and cyber incidents and ease of brute forcing or guessing easy passwords and to make a case about strong password policies.
