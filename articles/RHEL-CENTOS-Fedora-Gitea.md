# Gitea

## Postgresql configuration

PostgreSQL uses md5 challenge-response encryption scheme for password authentication by default. Nowadays this scheme is not considered secure anymore. Use SCRAM-SHA-256 scheme instead by editing the postgresql.conf configuration file on the database server to:

```
password_encryption = scram-sha-256
```

Restart PostgreSQL to apply the setting.


On the database server, login to the database console as superuser:

```
su -c "psql" - postgres
```

Create database user (role in PostgreSQL terms) with login privilege and password. Please use a secure, strong password instead of 'gitea' below:

```
CREATE ROLE gitea WITH LOGIN PASSWORD 'gitea';
```

Replace username and password as appropriate.

Create database with UTF-8 charset and owned by the database user created earlier. Any libc collations can be specified with LC_COLLATE and LC_CTYPE parameter, depending on expected content:

```
CREATE DATABASE giteadb WITH OWNER gitea TEMPLATE template0 ENCODING UTF8 LC_COLLATE 'en_US.UTF-8' LC_CTYPE 'en_US.UTF-8';
```

Replace database name as appropriate.


Allow the database user to access the database created above by adding the following authentication rule to pg_hba.conf.

For local database:

```
local    giteadb    gitea    scram-sha-256
```


For remote database:

```
host    giteadb    gitea    192.0.2.10/32    scram-sha-256
```

Replace database name, user, and IP address of Gitea instance with your own.

Note: rules on pg_hba.conf are evaluated sequentially, that is the first matching rule will be used for authentication. Your PostgreSQL installation may come with generic authentication rules that match all users and databases. You may need to place the rules presented here above such generic rules if it is the case.

Restart PostgreSQL to apply new authentication rules.

On your Gitea server, test connection to the database.

For local database:

```
psql -U gitea -d giteadb
```

For remote database:

```
psql "postgres://gitea@203.0.113.3/giteadb"
```

where gitea is database user, giteadb is database name, and 203.0.113.3 is IP address of your database instance.

You should be prompted to enter password for the database user, and connected to the database.


## Prepare the Gitea Environment

Create a user to run Gitea.

```
sudo adduser --system --shell /bin/bash --comment 'Git Version Control' --user-group --home-dir /home/git -m git
```

Create the required directory structure.

```
sudo mkdir -p /var/lib/gitea/{custom,data,indexers,public,log}
sudo chown git:git /var/lib/gitea/{data,indexers,log}
sudo chmod 750 /var/lib/gitea/{data,indexers,log}
sudo mkdir /etc/gitea
sudo chown root:git /etc/gitea
sudo chmod 770 /etc/gitea
```

## Install Gitea


Download the Gitea binary using the method on the official distribution page.

Copy the binary to a global location.

```
sudo cp gitea /usr/local/bin/gitea

sudo chown git:git /usr/local/bin/gitea
sudo chmod 750 /usr/local/bin/gitea
```

## Create a service file to start Gitea automatically

Create a linux service file.

```
sudo nano /etc/systemd/system/gitea.service
```

Using a text editor of your choice, open this newly create file and populate it with the following.

```
[Unit]
Description=Gitea (Git with a cup of tea)
After=syslog.target
After=network.target
###
# Don't forget to add the database service requirements
###
#
#Requires=mysql.service
#Requires=mariadb.service
#Requires=postgresql.service
#Requires=memcached.service
#Requires=redis.service
#
###
# If using socket activation for main http/s
###
#
#After=gitea.main.socket
#Requires=gitea.main.socket
#
###
# (You can also provide gitea an http fallback and/or ssh socket too)
#
# An example of /etc/systemd/system/gitea.main.socket
###
##
## [Unit]
## Description=Gitea Web Socket
## PartOf=gitea.service
##
## [Socket]
## Service=gitea.service
## ListenStream=<some_port>
## NoDelay=true
##
## [Install]
## WantedBy=sockets.target
##
###

[Service]
# Modify these two values and uncomment them if you have
# repos with lots of files and get an HTTP error 500 because
# of that
###
#LimitMEMLOCK=infinity
#LimitNOFILE=65535
RestartSec=2s
Type=simple
User=git
Group=git
WorkingDirectory=/var/lib/gitea/
# If using Unix socket: tells systemd to create the /run/gitea folder, which will contain the gitea.sock file
# (manually creating /run/gitea doesn't work, because it would not persist across reboots)
#RuntimeDirectory=gitea
ExecStart=/usr/local/bin/gitea web --config /etc/gitea/app.ini
Restart=always
Environment=USER=git HOME=/home/git GITEA_WORK_DIR=/var/lib/gitea
# If you want to bind Gitea to a port below 1024, uncomment
# the two values below, or use socket activation to pass Gitea its ports as above
###
#CapabilityBoundingSet=CAP_NET_BIND_SERVICE
#AmbientCapabilities=CAP_NET_BIND_SERVICE
###

[Install]
WantedBy=multi-user.target
```

Enable and start Gitea at boot.

```
sudo systemctl daemon-reload
sudo systemctl enable gitea
sudo systemctl start gitea
```

Ensure Gitea is running.

```
sudo systemctl status gitea
```

Enable traffic to Gitea's default port in firewalld:

```
sudo firewall-cmd --add-port 3000/tcp --permanent
sudo firewall-cmd --reload 
```

Finally, open a web browser and point it to:

```
http://YOUR_SERVER_IP:3000/install
```

Follow the on-screen instructions to complete the Gitea setup.


## Generate SSH Key

Generating a new SSH key
Open Terminal.

Paste the text below, substituting in your GitHub email address.

```bash
$ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

This creates a new ssh key, using the provided email as a label.

```bash
> Generating public/private rsa key pair.
```

When you're prompted to "Enter a file in which to save the key," press Enter. This accepts the default file location.

```bash
> Enter a file in which to save the key (/home/you/.ssh/id_rsa): [Press enter]
```

At the prompt, type a secure passphrase. For more information, see "Working with SSH key passphrases".

```bash
> Enter passphrase (empty for no passphrase): [Type a passphrase]
> Enter same passphrase again: [Type passphrase again]
```