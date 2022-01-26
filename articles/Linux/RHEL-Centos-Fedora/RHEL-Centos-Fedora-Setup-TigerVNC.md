# How to Install and Configure VNC Server on Centos 8 / RHEL 8

A VNC (Virtual Network Computing) Server is a GUI based desktop sharing platform that allows you to access remote desktop machines. In Centos 8 and RHEL 8 systems, VNC servers are not installed by default and need to be installed manually. In this article, we’ll look at how to install VNC Server on CentOS 8 / RHEL 8 systems with a simple step-by-step installation guide.

## Prerequisites to Install VNC Server on Centos 8 / RHEL 8

To install VNC Server in your system, make sure you have the following requirements readily available on your system:

* CentOS 8 / RHEL 8
* GNOME Desktop Environment
* Root access
* DNF / YUM Package repositories

## Step by Step Guide to Install VNC Server on Centos 8 / RHEL 8

### Step 1)  Install GNOME Desktop environment

Before installing VNC Server in your CentOS 8 / RHEL 8, make sure you have a desktop Environment (DE) installed. In case GNOME desktop is already installed or you have installed your server with gui option then you can skip this step.

In CentOS 8 / RHEL 8, GNOME is the default desktop environment. if you don’t have it in your system, install it using the following command:

```bash
   dnf groupinstall "workstation"

   OR

   dnf groupinstall "Server with GUI
```

Once the above packages are installed successfully then run the following command to enable the graphical mode

```bash
   systemctl set-default graphical
```

Now reboot the system so that we get GNOME login screen.

```bash
   reboot
```

Once the system is rebooted successfully uncomment the line **"WaylandEnable=false"** from the file **"/etc/gdm/custom.conf"** so that remote desktop session request via vnc is handled by xorg of GNOME desktop in place of wayland display manager.

>>>Note: Wayland is the default display manager (GDM) in GNOME and it not is configured to handled remote rendering API like X.org

### Step 2) Install VNC Server (tigervnc-server)

Next we'll install the VNC Server, there are lot of VNC Servers available, and for installation purposes, we'll be installing **TigerVNC Server**. It is one of the most popular VNC Server and a high-performance and platform-independent VNC that allows users to interact with remote machines easily.

Now install TigerVNC Server using the following command:

```bash
   dnf install tigervnc-server tigervnc-server-module -y
```

### Step 3) Set VNC Password for Local User

Let's assume we want 'maksym' user to use VNC for remote desktop session, then switch to the user and set its password using vncpasswd command,

```bash
   su - maksym
   vncpasswd

   Password:
   Verify:
   Would you like to enter a view-only password (y/n)? n
   A view-only password is not used
```

### Step 4) Setup VNC Server Configuration File

Next step is to configure VNC Server Configuration file. Create a file **"/etc/systemd/system/vncserver@.service"** with the following content so that tigervnc-server's service started for above local user **"maksym"**.

```bash
   nano /etc/systemd/system/vncserver@.service
```

```bash
   [Unit]
   Description=Remote Desktop VNC Service
   After=syslog.target network.target

   [Service]
   Type=forking
   WorkingDirectory=/home/maksym
   User=maksym
   Group=maksym

   ExecStartPre=/bin/sh -c '/usr/bin/vncserver -kill %i > /dev/null 2>&1 || :'
   ExecStart=/usr/bin/vncserver -autokill %i
   ExecStop=/usr/bin/vncserver -kill %i

   [Install]
   WantedBy=multi-user.target
```

Save and exit the file,

>>>Note: Replace the user name in above file which suits to your setup.

### Step 5) Start VNC Service and allow port in firewall

I am using display number as 1, so use the following commands to start and enable vnc service on display number "1",

```bash
   systemctl daemon-reload
   systemctl start  vncserver@:1.service
   systemctl enable  vncserver@:1.service
   Created symlink /etc/systemd/system/multi-user.target.wants/vncserver@:1.service → /etc/systemd/system/vncserver@.service.
```

Use below netstat or ss command to verify whether VNC server start listening its request on 5901,

```bash
   [root@linuxtechi ~]# netstat -tunlp | grep 5901
   tcp        0      0 0.0.0.0:5901            0.0.0.0:*               LISTEN      8169/Xvnc
   tcp6       0      0 :::5901                 :::*                    LISTEN      8169/Xvnc
   [root@linuxtechi ~]# ss -tunlp | grep -i 5901
   tcp   LISTEN  0       5                    0.0.0.0:5901           0.0.0.0:*      users:(("Xvnc",pid=8169,fd=6))                    
   tcp   LISTEN  0       5                       [::]:5901              [::]:*      users:(("Xvnc",pid=8169,fd=7))                    
   [root@linuxtechi ~]#
```

Use below systemctl command to verify the status of VNC server,

```bash
   systemctl status vncserver@:1.service
```

Above command’s output confirms that VNC is started successfully on port tcp port 5901. Use the following command allow VNC Server port “5901” in os firewall,

```bash
   firewall-cmd --permanent --add-port=5901/tcp
   success
   firewall-cmd --reload
   success
```

### Step 6) Connect to Remote Desktop Session

Now we are all set to see if the remote desktop connection is working. To access the remote desktop, Start the VNC Viewer from your Windows  / Linux workstation and enter your VNC server IP Address and Port Number and then hit enter.

Next, it will ask for your VNC password. Enter the password that you have created earlier for your local user and click OK to continue