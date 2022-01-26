# KMS Server

Make sure "build-essential" package is installed on your Debian/Ubuntu machine. Login to you Linux with SSH and follow steps below:

```bash
cd /opt/
git clone https://github.com/kebe7jun/linux-kms-server
useradd -s /usr/sbin/nologin -r -M vlmcsd
cd /opt/linux-kms-server/vlmcsd/
make
```

Wait for it to finish, if there are no errors/warning continue with following:

```bash
nano /lib/systemd/system/vlmcsd.service
```

Add following to file:

```bash
[Unit]
Description=vlmcsd KMS emulator service
After=network-online.target
Wants=network-online.target 

[Service]
Type=forking
User=vlmcsd
ExecStart=/opt/linux-kms-server/vlmcsd/vlmcsd -l /var/log/vlmcsd/vlmcsd.log 

[Install]
WantedBy=multi-user.target
```

Save the file and continue creating the log folder and configure the permissions:

```bash
mkdir /var/log/vlmcsd
chown vlmcsd:vlmcsd /var/log/vlmcsd
systemctl enable vlmcsd
systemctl start vlmcsd
```

To verify the status if the service is running, run following:

```bash
systemctl status vlmcsd
```

If everything is fine, it should look like this:

```bash
root@kms:~# systemctl status vlmcsd
? vlmcsd.service - vlmcsd KMS emulator service
Loaded: loaded (/lib/systemd/system/vlmcsd.service; enabled; vendor preset: enabled)
Active: active (running) since Sat 2020-05-02 22:33:16 CEST; 42s ago
Process: 457 ExecStart=/opt/linux-kms-server/vlmcsd/vlmcsd -l /var/log/vlmcsd/vlmcsd.log (code=exited, status=0/SUCCESS)
Main PID: 466 (vlmcsd)
Tasks: 1 (limit: 1059)
Memory: 396.0K
CGroup: /system.slice/vlmcsd.service
+-466 /opt/linux-kms-server/vlmcsd/vlmcsd -l /var/log/vlmcsd/vlmcsd.log
```

## Windows activation

Answer
NOTE: if you are not on the FSU network, you will need to use FSU's Virtual Private Network (VPN) Service . 

Right Click on the Start menu and select Command Prompt (Admin)
Run the command cscript slmgr.vbs -skms fsu-kms-01.fsu.edu to configure computer for the KMS activation server.
Run comand for the KMS activation server
Run the command cscript slmgr.vbs -ato to activate the computer with the KMS server.
Run comand KMS server
Finally run cscript slmgr.vbs -dli to display your license information.
You should see VOLUME_KMSCLIENT Channel
Run script for license info

## Office activation

To activate a product like Office 2016 against this KMS emulator, you can use the ospp.vbs script located in your Office installation folder:

```cmd
cd "C:\Program Files (x86)\Microsoft Office\Office16"
```

Set the KMS server to be used for activation:

```cmd
cscript ospp.vbs /sethst:#YOUR_IP_ADDRESS_HERE
```
 
Activate Office against the set server:

```cmd
cscript ospp.vbs /act
```