# Guacamole Debian 11 installation guide

## Mandatory packages

```bash
sudo apt install libcairo2-dev libjpeg62-turbo-dev libpng-dev libtool-bin libossp-uuid-dev -y
```

## Optional packages

```bash
sudo apt install libavcodec-dev libavformat-dev libavutil-dev libswscale-dev freerdp2-dev libpango1.0-dev libssh2-1-dev libtelnet-dev libvncserver-dev libwebsockets-dev libpulse-dev libssl-dev libvorbis-dev libwebp-dev -y
```

## Latest version install

```bash
git clone git://github.com/apache/guacamole-server.git
```

```bash
sudo apt install autotools-dev autoconf -y
```

```bash
cd guacamole-server
autoreconf -fi
```


Once you run configure, you can see what a listing of what libraries were found and what it has determined should be built:

./configure --with-init-dir=/etc/init.d

For debian 11 adapt Guacamole server to your system

```bash
./configure --with-systemd-dir=/etc/systemd/system/
```

Compile and install Guacamole Server on Debian 11;

```bash
sudo make
sudo make install
```

Next, run the ldconfig command to create the necessary links and cache to the most recent shared libraries found in the guacamole server directory.

```bash
ldconfig
```

Running Guacamole-Server on Debian 11
Reload systemd configuration files and start and enable guacd (Guacamole Daemon) to run on boot after the installation.
sudo systemctl daemon-reload
sudo systemctl enable --now guacd
To check the status;

```
sudo systemctl status guacd


## Install Tomcat Servlet
Apache Tomcat is used to serve guacamole client content to users that connects to guacamole server via the web browser. To install Tomcat, run the command below;

```bash
sudo apt install tomcat9 tomcat9-admin tomcat9-common tomcat9-user -y
```

Tomcat9 is started and enabled to run on system boot upon installation. Check the status by running the command below;

```bash
systemctl status tomcat9.service
```

Apache Tomcat listens on port 8080/tcp by default - allow 8080/tcp


## Install Guacamole Client on Debian 11

guacamole-client contains provides web application that will serve the HTML5 Guacamole client to users that connect to your server. The web application will then connect to guacd on behalf of connected users in order to serve them any remote desktop they are authorized to access.
Create Guacamole configuration directory;

```bash
mkdir /etc/guacamole
```

## Download Guacamole-client Binary
Guacamole client can be installed from source code or from ready binary. Binary installation is used in this demo.

```bash
VER=1.3.0
sudo wget https://guacamole.apache.org/releases/$VER/binary/guacamole-$VER.war -O /etc/guacamole/guacamole.war
```

Create a symbolic link of the guacamole client to Tomcat webapps directory as shown below;

```bash
sudo ln -s /etc/guacamole/guacamole.war /var/lib/tomcat9/webapps/
```

Restart Tomcat and Guacamole server to deploy the new web application;


```bash
sudo systemctl restart tomcat9 guacd
```

## Configure Apache Guacamole on Debian 11

Guacamole has two major configuration files;

* /etc/guacamole which is referenced by the GUACAMOLE_HOME environment variable
* /etc/guacamole/guacamole.properties which is the main configuration file used by Guacamole and its extensions.

There are also guacamole extensions and libraries configurations. You need to create the directories for these configs;

```bash
sudo mkdir /etc/guacamole/{extensions,lib}
```

Set the guacamole home directory environment variable and add it to /etc/default/tomcat9 configuration file.

```bash
sudo su
echo "GUACAMOLE_HOME=/etc/guacamole" >> /etc/default/tomcat9
```

## Configure Guacamole Server Connections

To define how Guacamole connects to guacd, create the guacamole.properties file under /etc/guacamole directory with the following content.

```bash
cat > /etc/guacamole/guacamole.properties << EOL
guacd-hostname: localhost
guacd-port: 4822
user-mapping:   /etc/guacamole/user-mapping.xml
auth-provider:  net.sourceforge.guacamole.net.basic.BasicFileAuthenticationProvider
EOL
```

Next, link the Guacamole configurations directory to Tomcat servlet directory as shown below.

```bash
sudo ln -s /etc/guacamole /usr/share/tomcat9/.guacamole
```

## Configure Guacamole Authentication Method

Guacamole’s default authentication method reads all users and connections from a single file called user-mapping.xml.

In this file,you need to define the users allowed to access Guacamole web UI, the servers to connect to and the method of connection.

Other authentication methods are supported, but beyond the scope of this tutorial.

To begin with, generate the MD5 hash of passwords for the user to be used for logging into Guacamole web user interface.

Replace your password accordingly;

```bash
sudo su
echo -n password | openssl md5
```

```bash
(stdin)= 12f0257f627333ee814fefc80756fa65
```

Be sure to replace password with your strong password.

Next, create the default user authentication file, user-mapping.xml with the following contents.

```bash
sudo nano /etc/guacamole/user-mapping.xml
```

If you dont specify the username and password in the file, you will be prompted to provide them while attempting to login, which i consider it abit secure.

If you need to explicitly define usernames and passwords in the configuration file, add the parameters;

```bash
<param name="username">USERNAME</param>
<param name="password">PASSWORD</param>
```

Save and exit the configuration file.

Restart both Tomcat and guacd to effect the changes.

```bash
sudo systemctl restart tomcat9 guacd
```

Be sure to check the syslog, /var/log/syslog or /var/log/tomcat9/ log files for any issues.

## Accessing Apache Guacamole from Browser

Apache Guacamole server is now setup. You can access it from web browser using the address 

```bash
http://maks-it.com:8080/guacamole
```

## Common issues and workarounds







### If you encounter CONNECTION ERROR, and upon checking the logs

```bash
tail -f /var/log/syslog
Sep 11 21:45:45 debian11 guacd[1109]: FreeRDP initialization may fail: The current user's home directory ("/usr/sbin") is not writable, but FreeRDP generally requires a writable home directory for storage of configuration files and certificates.
Sep 11 21:45:45 debian11 guacd[1109]: guacd[1109]: WARNING:#011FreeRDP initialization may fail: The current user's home directory ("/usr/sbin") is not writable, but FreeRDP generally requires a writable home directory for storage of configuration files and certificates.
Sep 11 21:45:45 debian11 guacd[1109]: No security mode specified. Defaulting to security mode negotiation with server.
Sep 11 21:45:45 debian11 guacd[1109]: guacd[1109]: INFO:#011No security mode specified. Defaulting to security mode negotiation with server.
Sep 11 21:45:45 debian11 guacd[1109]: Resize method: none
Sep 11 21:45:45 debian11 guacd[1109]: guacd[1109]: INFO:#011RDP server closed/refused connection: Security negotiation failed (wrong security type?)
```

Then fix it as follows:

Guacamole server (guacd) service runs as user daemon by default.

```bash
ps aux | grep -v grep| grep guacd
daemon       635  0.0  1.4 625480 14864 ?        Ssl  21:08   0:00 /usr/local/sbin/guacd -f
daemon       680  0.3  4.1 449468 41944 ?        Sl   21:09   0:08 /usr/local/sbin/guacd -f
daemon       804  0.0  3.9 359520 39488 ?        Sl   21:41   0:00 /usr/local/sbin/guacd -f
```

Create a guacd system user account which can be used to run guacd instead of running as daemon user.

```bash
useradd -M -d /var/lib/guacd/ -r -s /sbin/nologin -c "Guacd User" guacd
```

```bash
mkdir /var/lib/guacd
```

```bash
chown -R guacd: /var/lib/guacd
```

Next, update the Guacd service user:

```bash
sed -i 's/daemon/guacd/' /etc/systemd/system/guacd.service
```

Reload systemd daemon:

```bash
systemctl daemon-reload
```

Restart Guacd Service:

```bash
systemctl restart guacd
```

### Windows 10 and Windows Server 2016 RDP

```bash
[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp]
```

Change “SecurityLayer” value to dword:00000001
Verify “UserAuthentication” value is dword:0x00000000

This should work without reboot

Or use NLA

If you want to use NLA you have to set SecurityMode to NLA of the connection. And you have to set username and password!

If your username and password in guacamole are the same on windows machine (if you have active directory and ldap auth in guacamole) you can use:

Username: ${GUAC_USERNAME}
Password: ${GUAC_PASSWORD}
