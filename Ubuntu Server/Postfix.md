Postfix on the Ubuntu Server will act as our Mail Transfer Agent (MTA). In a typical corp environment, everything related to email would be managed and maintained by a third-party, like Microsoft or Google. This is the SaaS. Why are however provisioning an Email Server to understand how it works and why we need to protect it.

![[Drawing 2025-03-10 15.01.44.excalidraw.svg]]
> [!Note] End to End Email
> We will not be creating an end to end email program from `johnd` to `janed` as it requires creating email addresses and other network resources. For this purpose, we will only work with email between the Postfix server and `janed`'s client.

## Security Implications
### Common Threats
#### Open Relay Attacks
If improperly configured, your server can be used by spammers to send large volumes of email, damaging your IP reputation.
#### Brute Force Attacks
Attackers often attempt to compromise accounts via brute force or credential stuffing
#### Spam and Phishing
Attackers may spoof your domain or use your server for phishing campaigns
#### Malware Delivery
Your server could inadvertently become a vehicle for spreading malware if attachments are not scanned.
## Setup Postfix
If you are still logged in the CORP account of the email server, we need to first logout and go back to the local account. You can simply use the `exit` command in the terminal to get you back.
### Install Postfix
Make sure the server is up to date before proceeding. To install Postfix, we will use the following command:

```bash
sudo DEBIAN_PRIORITY=low apt install postfix
```

We are using `DEBIAN_PRIORITY=low` to allow the installation to prompt us while installing. Hit `Enter` and press `y` when prompted. You will be presented with the **Postfix Configuration** screen.
![[Screenshot from 2025-03-10 15-16-51.png]]
Keep the default selection and proceed. Next is the **System mail name**, if it is already input as `email-svr` leave it as it is and proceed.
![[Screenshot from 2025-03-10 15-18-15.png]]
Use the same name for root and postmaster mail.
![[Screenshot from 2025-03-10 15-19-35.png]]
Pass through the remaining steps without changing any values to finish the installation.
## Configuration
### Set the Ubuntu User's Mailbox
This will define the default mailbox for the user.
```bash
sudo postconf -e 'home_mailbox = Maildir/'
```

We will also map the email accounts to the Linux system accounts
```bash
sudo postconf -e 'virtual_alias_maps= hash:/etc/postfix/virtual'
```

To map the accounts, create a new file
```bash
sudo nano /etc/postfix/virtual
```

We will create the email IDs we want in this file and map them to `email-svr`
```
contact@corp.pilgrimcorp-dc.com email-svr
admin@corp.pilgrimcorp-dc.com email-svr
email-svr@corp.pilgrimcorp-dc.com email-svr
```

To create the mapping, use the following command
```bash
sudo postmap /etc/postfix/virtual
```

Finally, to apply the configuration, we will restart the `postfix` service like this
```bash
sudo systemctl restart postfix
```
### Configure Uncomplicated Firewall to `allow` traffic
```bash
sudo ufw allow Postfix
```

You should see an output like this
```
Rules updated
Rules updated (v6)
```

Now, let's enable the firewall
```bash
sudo ufw enable
```

To verify if Postfix will be allowed in the firewall, use this command
```bash
sudo ufw app list
```

You should see Postfix in the output.
## Install S-Nail Email Client
### Export Environment Variable
We will export our mailbox directory as an environment variable in our shell environment
```bash
echo 'export MAIL=~/Maildir' | sudo tee -a /etc/bash.bashrc | sudo tee -a /etc/profile.d/mail.sh
```

Add the variable into the current session
```bash
source /etc/profile.d/mail.sh
```

Finally we will install the mail client S-nail
```bash
sudo apt install s-nail mailutils
```
### Configure S-Nail
```bash
sudo nano /etc/s-nail.rc
```

Add the following configuration at the end of the file
```rc
set emptystart
set folder=Maildir
set record=+sent
```
### Change Hostname
For this email server to function correctly, we need to change the hostname present in the `hostname` file.
```bash
sudo nano /etc/hostname
```

It will already have a value of `email-svr`. Change this to `smtp.corp.pilgrimcorp-dc.com`
## Postfix Configuration
To configure Postfix, we need to make changes in the Postfix configuration file
```bash
sudo nano /etc/postfix/main.cf
```

Use the arrow key to go all the way down. We need to make changes from the `smtpd_relay_restrictions` block.
```config
myhostname = smtp.corp.pilgrimcorp-dc.com
mydomain = corp.pilgrimcorp-dc.com
mydestination = $myhostname, localhost.localdomain, localhost
mynetworks = 127.0.0.0/8 10.0.0.0/24 [::ffff:127.0.0.0]/104 [::1]/128
```

Save the file and exit. Now it's time to apply the changes by restarting the postfix server.
```bash
sudo systemctl restart postfix
```
## Add the SMTP Record in Domain Controller
In the Domain Controller, go to **Server Manager** > **Tools** > **DNS**.
Click on the server then **Forward Lookup Zones** and then our right click Domain **corp.pilgrimcorp-dc.com**
Click on **New Host (A or AAAA)** and use these settings
![[Screenshot from 2025-03-10 18-24-42.png]]
## Create the Maildir Directory
Let's create the `Maildir` directory in the home folder of the user. Make sure you are in the Home folder using `pwd` command.

```bash
email-svr@smtp:~$ pwd
/home/email-svr
email-svr@smtp:~$ mkdir Maildir
email-svr@smtp:~$ ls
Maildir
```

Now we will send a test mail using S-nail
```bash
email-svr@smtp:~$ echo 'init' | s-nail -s 'init' -Snorecord email-svr
```

To verify that you have received the email, use this command
```bash
email-svr@smtp:~$ s-nail
```

As we can see, we have 1 email
![[Screenshot from 2025-03-10 18-31-32.png]]

Type 1 next to the question mark (meaning the number of the email you want to see) and press `Enter` to see the email like you are used to seeing.
![[Screenshot from 2025-03-10 18-32-39.png]]

We have verified that our email configuration works correctly. Let's delete the test email as it is not necessary.
```bash
? d 1
```

Finally, press `q` and `Enter` to exit out.

Now, let's try and send an email as another account.
```bash
email-svr@smtp:~$ nano test_message
```

Write the text `Hello! This is a test!` and save the file.

Now, let's send this message using s-nail.
```bash
email-svr@smtp:~$ cat ~/text_message | s-nail -s 'Test Message' -r janed@corp.pilgrimcorp-dc.com email-svr@smtp.corp.pilgrimcorp-dc.com
```

Now if we check s-nail again, we should have 1 email but sent from `janed`.
![[Screenshot from 2025-03-10 18-42-06.png]]

This completes our entire Postfix setup. And here we will take another snapshot. and call it `base+postfix`.