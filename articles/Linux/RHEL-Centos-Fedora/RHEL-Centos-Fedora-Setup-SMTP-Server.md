We are going to set up an Email server using Postfix, Dovecot and Squirrelmail on CentOS 7.x. We will be using Postfix for SMTP (Simple Mail Transfer Protocol), Dovecot for POP/IMAP and Squirrelmail as webmail client to send or receive emails. We will also setup MX records which is important to route the emails.

# Requirements

The requirements for setting up email server will simply be a VPS or dedicated server with a fresh CentOS 7.x install and also a static IP address. In this tutorial we will be using a root account to execute the commands. If you are logged in as non-root user, use sudo command at the start of all the commands or you can execute su command to login as root user.

# Setting up DNS

It is very important to setup DNS records, specifically MX records in your domain control panel. Login to your domain control panel and change your DNS settings to add these following MX records entries. A typical MX record will look like this.
```
srvmta0001      IN  A   192.168.0.5
@        IN MX  10      srvmta0001.nc.local.

```

Where MX is the type of record, MX stands for Mail Exchangers. Next is the value for host, you can either enter your domain name or you can also use @ which represents the value of the zone name which is same as your domain name. Next you will have to choose the destination, you will need to enter the hostname or FQDN of your mail server.

The next value is the priority. The lowest number is priority. For example 0 will have the highest priority and 20 will have a lower priority. Priority is used because we can add multiple MX records for a single domain, mail is forwarded to the server having highest priority. If the server having highest priority is not available then mail will be forwarded to the server having second highest priority. Next is TTL or Time to Live, it should be set to 3600.

It is very important that you also setup an A record for your hostname of the mail server FQDN. Again select the type as A record, host should be the hostname you are using in your FQDN, for example in this case we have used the hostname as mail.yourdomain.com, hence our host will be mail. Next, at destination, enter the IP address of your server. A records does not have priority option hence, you will only need to provide TTL.

Once you have configured your DNS settings, you will need to wait some time so that DNS gets propagated. It usually takes around two hours these days. Once propagated, you can check your MX records [here](http://mxtoolbox.com/).

Until your DNS gets propagated, you can continue with the installation.

# Preparation
Login to your server and run the following command to update the repository and packages available in your system.
```
yum -y update
```

Now update the hostname of your system to the FQDN you want to use with your mail server. Run the following command to change your hostname.
```
nano /etc/hostname
```

Add following line:
```
hostname srvmta0001.nc.local
```

You can replace the hostname according to your choice, but it should be same as the FQDN which we have used in our DNS settings.

Now add the hostname entry in the hosts file:

```
nano /etc/hosts
```

You will see two lines of entries in there, append your server IP address followed by hostname at the end of the file. It should look like the one shown below.

```
127.0.0.1 localhost localhost.localdomain localhost4 localhost4.localdomain4
::1       localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.0.5 srvmta0001.nc.local
```

# Installing Postfix & Dovecot
Now we can install Postfix and Dovecot, enter the following commands to do so.
```
yum -y install postfix
yum -y install dovecot
```

# Configure Postfix

## SSL certificate creation
Before configuring postfix we will need to configure SSL which will be used to encrypt and secure the emails.

Now we will have to create SSL certificates. If you do not have openssl installed, install it with following command:
```
yum -y install openssl
```

Create the folder which will contain our generated certificate and key:
```
mkdir /etc/postfix/ssl
cd /etc/postfix/ssl
```

Now run command to create certificate and key files:
```
openssl req -x509 -nodes -newkey rsa:2048 -keyout server.key -out server.crt -nodes -days 365
```

Alternative:
```
openssl genrsa -des3 -out server.key 2048
openssl rsa -in server.key -out server.key.insecure
mv server.key server.key.secure
mv server.key.insecure server.key
```
Leave blank for **A challenge password []** value in the below step
```
openssl req -new -key server.key -out server.csr
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
```

You will be asked some information which is to be added into your CSR (Code Signing Request). You will be asked your country name in two letters, for example consider IN for India. Then you will be asked about the state or province. Then you will be asked about your city and organization. Finally, a common name of your server and your email address. If you want to leave some details blank use full stop or period ( . ) sign. You can also enter the default values just by pressing enter. Example output is given below.

```
Generating a 2048 bit RSA private key
..........................+++
...........................+++
writing new private key to 'server.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:IT
State or Province Name (full name) []:MN
Locality Name (eg, city) [Default City]:
Organization Name (eg, company) [Default Company Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (eg, your name or your server's hostname) []:
Email Address []:
```

This will generate the key file and certificates and will save then in /etc/postfix/ssl directory.

## Continue with Postfix configuration
Now edit postfix configuration file which can be found at /etc/postfix/main.cf:
```
nano /etc/postfix/main.cf
```

and append these lines at the end of the file.
```
myhostname = srvmta001.nc.local
mydomain = nc.local
myorigin = $mydomain
home_mailbox = mail/
mynetworks = 127.0.0.0/8
inet_interfaces = all
inet_protocols = all
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
smtpd_sasl_type = dovecot
smtpd_sasl_path = private/auth
smtpd_sasl_local_domain =
smtpd_sasl_security_options = noanonymous
broken_sasl_auth_clients = yes
smtpd_sasl_auth_enable = yes
smtpd_recipient_restrictions = permit_sasl_authenticated,permit_mynetworks,reject_unauth_destination
smtp_tls_security_level = may
smtpd_tls_security_level = may
smtp_tls_note_starttls_offer = yes
smtpd_tls_loglevel = 1
smtpd_tls_key_file = /etc/postfix/ssl/server.key
smtpd_tls_cert_file = /etc/postfix/ssl/server.crt
smtpd_tls_received_header = yes
smtpd_tls_session_cache_timeout = 3600s
tls_random_source = dev:/dev/urandom
```

Open another configuration file /etc/postfix/master.cf:
```
nano  /etc/postfix/master.cf
```

find the following lines in the configuration file:
```
# service type  private unpriv  chroot  wakeup  maxproc command + args
#               (yes)   (yes)   (yes)   (never) (100)
# ==========================================================================
smtp      inet  n       -       n       -       -       smtpd
```

Now add the following lines at just below these lines.

```
submission     inet  n       -       n       -       -       smtpd
  -o syslog_name=postfix/submission
  -o smtpd_sasl_auth_enable=yes
  -o smtpd_recipient_restrictions=permit_sasl_authenticated,reject
  -o milter_macro_daemon_name=ORIGINATING
smtps     inet  n       -       n       -       -       smtpd
  -o syslog_name=postfix/smtps
  -o smtpd_sasl_auth_enable=yes
  -o smtpd_recipient_restrictions=permit_sasl_authenticated,reject
  -o milter_macro_daemon_name=ORIGINATING
```

Now check the configuration using **postfix check** command.


# Configure Dovecot
Once Dovecot is installed, edit the following file using your favorite editor.
```
nano /etc/dovecot/conf.d/10-master.conf
```

and find the following lines:
```
  # Postfix smtp-auth
```

Now Append the following lines, just below these lines:
```
# Postfix smtp-auth
#unix_listener /var/spool/postfix/private/auth {
#  mode = 0666
#}
  
unix_listener /var/spool/postfix/private/auth {
    mode = 0660
    user = postfix
    group = postfix
}

```

Now open another configuration using the following command.

```
nano /etc/dovecot/conf.d/10-auth.conf
```

and find the following lines.

```
# Space separated list of wanted authentication mechanisms:
#   plain login digest-md5 cram-md5 ntlm rpa apop anonymous gssapi otp skey
#   gss-spnego
# NOTE: See also disable_plaintext_auth setting.
auth_mechanisms = plain
```

Append login at the end of the line auth_mechanisms = plain to make it look like
```
# Space separated list of wanted authentication mechanisms:
#   plain login digest-md5 cram-md5 ntlm rpa apop anonymous gssapi otp skey
#   gss-spnego
# NOTE: See also disable_plaintext_auth setting.
auth_mechanisms = plain login
```

Again edit /etc/dovecot/conf.d/10-mail.conf file using your favorite editor.
```
nano /etc/dovecot/conf.d/10-mail.conf
```

Find the following lines
```
# 
#
#mail_location =
```

Now add the following line just below these lines:
```
mail_location = maildir:~/mail
```

Now edit /etc/dovecot/conf.d/20-pop3.conf using your favorite editor.
```
nano /etc/dovecot/conf.d/20-pop3.conf
```

and find the following lines.
```
#pop3_uidl_format = %08Xu%08Xv
```

Uncomment the above line to make it look like as shown below.
```
# Note that Outlook 2003 seems to have problems with %v.%u format which was
# Dovecot's default, so if you're building a new server it would be a good
# idea to change this. %08Xu%08Xv should be pretty fail-safe.
#
pop3_uidl_format = %08Xu%08Xv 
```

# Start Postfix & Dovecot services
Now restart postfix, and dovecot using the following command.
```
systemctl restart postfix
systemctl enable postfix
systemctl restart dovecot
systemctl enable dovecot
```

# Firewall configuration

Now if you have a firewall running you will need to allow port number 25, 587, 465, 110, 143, 993, 995 and 80. All the ports except 80 are used to send and receive emails and port 80 is used to make HTTP connections. HTTP connections will be used to access Squirrelmail using web interface.

To unblock all these ports from firewall, run the following commands.

```
firewall-cmd --permanent --add-service=smtp
firewall-cmd --permanent --add-port=587/tcp
firewall-cmd --permanent --add-port=465/tcp
firewall-cmd --permanent --add-port=110/tcp
firewall-cmd --permanent --add-service=pop3s
firewall-cmd --permanent --add-port=143/tcp
firewall-cmd --permanent --add-service=imaps
firewall-cmd --permanent --add-service=http
firewall-cmd --reload
```

If you do not want to use web mail client Squirrelmail, just remove the http rule
```
firewall-cmd --permanent --remove-service=http
firewall-cmd --reload
```

# Testing Postfix and Dovecot

We need locally installed telnet to be able to execute our configuration test

If you do not have telnet installed, then you can run the following command to install telnet.
```
yum -y install telnet
```

Start to type the following command in your terminal.
```
telnet srvmta0001.nc.local smtp
```

Once you are connected using telnet you will see following output.
```
Trying ::1...
Connected to localhost.
Escape character is '^]'.
220 srvmta0001.nc.local ESMTP Postfix
```

Now you can also send email using telnet. Use the following command to enter the sender username.
```
mail someotheremail@gmail.com
Subject: test email from postfix
this is a test
.
EOT
```

To test Dovecot, enter the following command.
```
telnet srvmta0001.nc.local pop3
```

You will see following output,
```
Trying 192.168.0.5...
Connected to srvmta0001.nc.local.
Escape character is '^]'.
+OK Dovecot ready.
```

It tells that Dovecot is working fine, you can login to your mail account by providing login command, then use pass command to enter your password. To view the mails in your account, use retr command.

```
user maksym
+OK
pass Password
+OK Logged in.
retr
-ERR There's no message 1.
quit
+OK Logging out.
Connection closed by foreign host.
```

Now create email users, run the following command to add a user.
Create user with **/sbin/nologin** shell to restrict login access.
```
useradd -m mailuser -s /sbin/nologin
passwd mailuser
```

The above command will add a new user liptan and the attribute -s /sbin/nologin will deny login using SSH. Last command will create a password for the new user.


# Installing Squirrelmail
As we have both Postfix and Dovecot working, we can now install Squirrelmail to your server. Squirrelmail does not comes with the default CentOS repository, hence you will need to add EPEL repository into your system using the following command.

```
yum -y install epel-release
```

Now you can install Squirrelmail using the following command.
```
yum -y install squirrelmail
```

After installing Squirrelmail you can configure it by running the configuration script.
```
cd /usr/share/squirrelmail/config/
./conf.pl
```

You will see following output.
```
SquirrelMail Configuration : Read: config.php (1.4.0)
---------------------------------------------------------
Main Menu --
1.  Organization Preferences
2.  Server Settings
3.  Folder Defaults
4.  General Options
5.  Themes
6.  Address Books
7.  Message of the Day (MOTD)
8.  Plugins
9.  Database
10. LanguagesD.  Set pre-defined settings for specific IMAP serversC   Turn color off
S   Save data
Q   Quit
Command >>
```

In Option 1 you can change your organisation preferences. It is recommended to change it according to your organisation in production environment. If you choose option 1, you will see following output.

```
SquirrelMail Configuration : Read: config.php (1.4.0)
---------------------------------------------------------
Organization Preferences
1.  Organization Name      : SquirrelMail
2.  Organization Logo      : ../images/sm_logo.png
3.  Org. Logo Width/Height : (308/111)
4.  Organization Title     : SquirrelMail $version
5.  Signout Page           : 
6.  Top Frame              : _top
7.  Provider link          : http://squirrelmail.org/
8.  Provider name          : SquirrelMailR   Return to Main Menu
C   Turn color off
S   Save data
Q   Quit
Command >>
```

Change the organisation name, logo and title according to your need. Once done, return to main menu using R command. In main menu choose option 2 for Server settings.

```
SquirrelMail Configuration : Read: config.php (1.4.0)
---------------------------------------------------------
Server SettingsGeneral
-------
1.  Domain                 : localhost
2.  Invert Time            : false
3.  Sendmail or SMTP       : SendmailA.  Update IMAP Settings   : localhost:143 (uw)
B.  Change Sendmail Config : /usr/sbin/sendmailR   Return to Main Menu
C   Turn color off
S   Save data
Q   Quit
Command >> 1
```

Change your domain name by selecting option 1.
```
The domain name is the suffix at the end of all email addresses.  If for example, your email address is jdoe@example.com, then your domain would be example.com.
[localhost]: srvmta0001.nc.local
```

Now change your MTA by selecting the 3rd option.
```
SquirrelMail Configuration : Read: config.php (1.4.0)
---------------------------------------------------------
Server SettingsGeneral
-------
1.  Domain                 : nc.local
2.  Invert Time            : false
3.  Sendmail or SMTP       : SendmailA.  Update IMAP Settings   : localhost:143 (uw)
B.  Change Sendmail Config : /usr/sbin/sendmailR   Return to Main Menu
C   Turn color off
S   Save data
Q   QuitCommand >> 3You now need to choose the method that you will use for sending
messages in SquirrelMail.  You can either connect to an SMTP server
or use sendmail directly.
1. Sendmail 2. SMTP Your choice [1/2] [1]: 2
```

Now save your setting by giving S command and finally quit using Q command.

Now you will need to install the Apache web server, so that we can access Squirrelmail using web interface. Run the following command to install Apache web server.

```
yum -y install httpd
```

Once Apache is installed, edit the configuration file to add a new virtual host.
```
nano /etc/httpd/conf/httpd.conf
```

Now add the following lines at the end of the file.
```
Alias /webmail /usr/share/squirrelmailOptions Indexes FollowSymLinks
RewriteEngine On
AllowOverride All
DirectoryIndex index.php
Order allow,deny
Allow from all
```


Save the file and start and enable Apache web server using the following commands.
```
systemctl start httpd
systemctl enable httpd
```

At this point you can browse Squirrelmail by going to following link into the browser.

```
http://webmail
```

You will see following screen.


Once you login you will see the following webmail interface.


You can now read your emails and send emails through this interface.



# Conclusion
We have installed an email server, using Postfix, Dovecot and the Squirrelmail webmail client. You can now successfully deploy the email server and start sending via the mail server.