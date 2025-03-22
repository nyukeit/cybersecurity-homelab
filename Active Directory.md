While Windows clients can connect native to Active Directory servers, our Unix based systems cannot. For this we will be using Samba + Winbind to achieve the AD connection.
## Security Implications of Active Directory
Active Directory is a very high value target and misconfigurations can expose sensitive data and enable risk of lateral movement and privilege escalation.
### Common Security Threats
#### Authentication
Weak passwords, techniques like Pass-the-Hash or Kerberoasting can allow attackers to compromise credentials.
#### Privilege Escalation
Over-permission-ed accounts may grant unnecessary access. Admin accounts are high-value targets.
#### Lateral Movement
Once inside, attackers can move through the network using AD to identify valuable targets, look for shared passwords and use compromised credentials to move into other accounts.