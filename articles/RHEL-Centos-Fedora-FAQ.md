# CentOS 7 FAQ

## 1. CentOS 7.x mirror

``` html
http://mirror.centos.org/centos/7/os/x86_64/
```

## 2. Get user and kill by port

This will print you PID of process bound on that port.

``` bash
fuser 8080/tcp
```

and this will kill that process.

``` bash
fuser -k 8080/tcp
```

More universal is

``` bash
lsof -i4
```

or for IPv6

``` bash
lsof -i6
```

## 3. Run VSCode as root

``` bash
sudo code --user-data-dir="~/.vscode"
```

## 4. Freeing disk space on your Linux server

The websites that I host on Slicehost, Playmary and Street Hoarding, keep crashing because my slice keeps running out of disk space.

To find out where disk space is being used:

1. Get to the root of your machine by running cd /
2. Run sudo du -h --max-depth=1
3. Note which directories are using a lot of disk space.
4. cd into one of the big directories.
5. Run ls -l to see which files are using a lot of space. Delete any you don’t need.
6. Repeat steps 2 to 5.

## 5. How to clean files under /var/spool/abrt

All crash report and back trace from kernel driver is written on subdirectories of /var/spool/abrt directory.

To permanently stop this back trace gathering, we need to stop below two processes.

```bash
# service abrtd stop
```

```bash
# service abrt-oops stop
```

And we can remove all those directories and files with following rm command:

```bash
# abrt-cli rm /var/spool/abrt/*
```

## 6. Resize XFS partition

1. alter your partition table so sda2 ends at end of disk
2. reread the partition table (will require a reboot)
3. resize your LVM pv using pvresize

### Step 1 - Partition table

1. Run fdisk /dev/sda. Issue p to print your current partition table and copy that output to some safe place. Now issue d followed by 2 to remove the second partition. Issue n to create a new second partition. Make sure the start equals the start of the partition table you printed earlier. Make sure the end is at the end of the disk (usually the default).
2. Issue t followed by 2 followed by 8e to toggle the partition type of your new second partition to 8e (Linux LVM).
3. Issue p to review your new partition layout and make sure the start of the new second partition is exactly where the old second partition was.

If everything looks right, issue w to write the partition table to disk. You will get an error message from partprobe that the partition table couldn't be reread (because the disk is in use).

### Step 2 - Reboot your system

This step is neccessary so the partition table gets re-read.

### Step 3 - Resize the LVM PV

After your system rebooted invoke pvresize /dev/sda2. Your Physical LVM volume will now span the rest of the drive and you can create or extend logical volumes into that space.

## Delete KDE Wallet

In order to delete the wallet, delete the file

```bash
/home/<user name>/.kde/share/apps/kwallet/kdewallet.kwl
```

## Firewall rules

```bash
   firewall-cmd --permanent --add-port=5901/tcp
   success
   firewall-cmd --reload
   success
```

```bash
   firewall-cmd --zone=public --remove-port=5901/tcp
   success
   firewall-cmd --runtime-to-permanent
   success
   firewall-cmd --reload
```

## If you have deleted root folder

So you deleted /root and now you have -bash-4.1# in your command line huh?

Well this is caused of a missing / corrupt .bashrc (in your case missing) file in /root (.bashrc sources /etc/bashrc which is what sets the prompt). To fix it, you will run the following command which runs when an account is created. Run as the root user (since it is the user having the problem) or you can define the destination path.

command: (make sure you are in /root)

cp -v /etc/skel/.bash* ~/
Exit terminal and log back in.

## Correct .ssh folder permissions

generate-ssh-key.sh

```bash
ssh-keygen -t rsa -b 4096 -N '' -C "rthijssen@gmail.com" -f ~/.ssh/id_rsa
ssh-keygen -t rsa -b 4096 -N '' -C "rthijssen@gmail.com" -f ~/.ssh/github_rsa
ssh-keygen -t rsa -b 4096 -N '' -C "rthijssen@gmail.com" -f ~/.ssh/mozilla_rsa
```

ssh-key-add.sh

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
ssh-add ~/.ssh/github_rsa
ssh-add ~/.ssh/mozilla_rsa
```

ssh-key-permissions.sh

```bash
chmod 700 ~/.ssh
chmod 644 ~/.ssh/authorized_keys
chmod 644 ~/.ssh/known_hosts
chmod 644 ~/.ssh/config
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
chmod 600 ~/.ssh/github_rsa
chmod 644 ~/.ssh/github_rsa.pub
chmod 600 ~/.ssh/mozilla_rsa
chmod 644 ~/.ssh/mozilla_rsa.pub
```

## Download all files from repository

Use git checkout. In your case:

```bash
git checkout origin/master style.css
```

This command will update the requested file from the given branch (here the remote branch origin/master).

## git push origin master everything up-to-date

```bash
   git commit
```

## What to do with commit made in a detached head

Create a branch where you are, then switch to master and merge it:

```bash
git branch my-temporary-work
git checkout master
git merge my-temporary-work
```

## Switch to text or graphical user interface

Graphical user interface:

```bash
systemctl set-default graphical.target
```

Text user interface:
```bash
systemctl set-default multi-user.target
```



## RPM

```bash
rpm -qa | grep -i webmin
```

You should then find the exact package name. Then run to remove

```bash
rpm -e <package name>
```


## Gnome Boxes

Guest additions
https://www.spice-space.org/download.html


Virtual Machine Manager Powered by libvirt
https://virt-manager.org/

## Kernel headers

```bash
/lib/modules
```

You can install the correct kernel header files like so:

```bash
$ sudo yum install "kernel-devel-uname-r == $(uname -r)"
```

This command will always install the right version.

```bash
$ sudo yum install "kernel-devel-uname-r == $(uname -r)"
Loaded plugins: auto-update-debuginfo, changelog, langpacks, refresh-packagekit
No package kernel-devel-uname-r == 3.12.6-200.fc19.x86_64 available.
Error: Nothing to do
```

Or you can search for them like this:

```bash
$ yum search "kernel-headers-uname-r == $(uname -r)" --disableexcludes=all
Loaded plugins: auto-update-debuginfo, changelog, langpacks, refresh-packagekit
Warning: No matches found for: kernel-headers-uname-r == 3.12.6-200.fc19.x86_64
No matches found
```

However I've notice this issue as well where specific versions of headers are not present in the repositories. You might have to reach into Koji to find a particular version of a build.

Information for build [kernel-5.6.8-200.fc31](https://koji.fedoraproject.org/koji/buildinfo?buildID=1499538)

That page includes all the assets for that particular version of the Kernel.


## VMWare install

To successfully install the product perform the following steps.

​1. If you have already attempted to install Workstation, then perform the uninstall as described in the product documentation: 

```bash
​​​vmware-installer -u vmware-workstation
```

2. Install the elfutils-libelf-devel package via terminal. On Fedora, this is accomplished with the following command or similar: 

```bash
sudo dnf install elfutils-libelf-devel
```

3. The VMMON and VMNET kernel modules are built on the first launch of Workstation. These modules require your Linux distribution's kernel development and header packages to be available or the build will fail. Before launching workstation ensure that these packages are installed with the following command on Fedora or similar: 
​
sudo yum install kernel-headers-`uname -r` kernel-devel-`uname -r`

4. Proceed with installation of Workstation for Linux.
 
Note: Workstation will attempt to auto detect the path to the kernel development and header packages at the end of the product installation. If you successfully installed the packages and they are not found please see your distros documentation for the location of these files.

5. If your system is configured with Secure Boot then the kernel modules must be signed before they will load or Virtual machines will not power on in Workstation. You will receive the following error: 

```bash
Cannot open /dev/vmmon: No such file or directory. Please make sure that the kernel module `vmmon' is loaded
```

    If you encounter this error after the modules have been successfully built, please see the following KB article for instructions on signing the modules for systems with Secure Boot enabled: https://kb.vmware.com/s/article/2146460

If you encounter further difficulties, please open a case with our teams to assist:
https://www.vmware.com/support/file-sr.html





Patch maybe....
https://github.com/mkubecek/vmware-host-modules

### Secure boot
On Linux host with secure mode enabled, it is not allowed to load any unsigned drivers. Due to this, VMware drivers, such as vmmon and vmnet, are not able to be loaded which prevents virtual machine to power on.

1. Generate a key pair using the openssl to sign vmmon and vmnet modules:

```bash
openssl req -new -x509 -newkey rsa:2048 -keyout MOK.priv -outform DER -out MOK.der -nodes -days 36500 -subj "/CN=VMware/"
```

2. Sign the modules using the generated key by running these commands:

```bash
sudo usr/src/kernels/`uname -r`/scripts/sign-file sha256 ./MOK.priv ./MOK.der $(modinfo -n vmmon)
```

```bash
sudo /usr/src/linux-headers-`uname -r`/scripts/sign-file sha256 ./MOK.priv ./MOK.der $(modinfo -n vmnet)
```

3. Import the public key to the system's MOK list by running this command:

```bash
mokutil --import MOK.der
```

* Confirm a password for this MOK enrollment request.
* Reboot your machine. Follow the instructions to complete the enrollment from the UEFI console.

## Execute files in linux

```bash
chmod a+x VMware-Player-6.0.3-1895310.x86_64.bundle
```

```bash
sudo ./VMware-Player-6.0.3-1895310.x86_64.bundle
```


## Gnome GUI network configuration utility

There is a hidden method to share your WiFi over Ethernet in the latest Gnome. I stumbled upon this while trying to connect my RaspberryPi 3B with my University’s Internet.

Type nm-connection-editor in your terminal.

```bash
sudo nm-connection-editor
```

Add a shared network connection by pressing the Add button.
Choose Ethernet from the list and press Create.
Click IPv4 Settings in the left.
Choose Shared to other computers by clicking the Method drop-down menu.
Enter a new name like Shared WiFi LAN as the Connection name at the top


## Create boot media in terminal

Creating Bootable CentOS 7 USB Stick on Linux
While there are many different GUI tools that allows you to flash ISO images to USB drives, in this tutorial, we will create a bootable CentOS 7 USB stick using the dd command.
Creating Bootable CentOS 7 USB Stick on Linux is a quick and easy process, just follow the steps detailed below.

Insert the USB flash drive into the USB port.

Find out the name of your USB drive with the lsblk command:

```bash
lsblk
```

The output will look like this:

```bash
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda           8:0    0 465.8G  0 disk 
└─sda1        8:1    0 465.8G  0 part /data
sdx           8:16   1   7.5G  0 disk 
└─sdx1        8:17   1   7.5G  0 part /run/media/linuxize/Kingston
nvme0n1     259:0    0 232.9G  0 disk 
├─nvme0n1p1 259:1    0   512M  0 part /boot
├─nvme0n1p2 259:2    0    16G  0 part [SWAP]
└─nvme0n1p3 259:3    0 216.4G  0 part /
```

In our case the name of the USB device is /dev/sdx but this may vary on your system.

On most Linux distributions the USB flash drive will be automatically mounted when inserted. Before flashing the image make sure the USB device is not mounted. To do so use the umount command followed by either the directory where it has been mounted (mount point) or the device name:

```bash
sudo umount /dev/sdx1
```

The last step is to flash the CentOS ISO image to the USB drive. Make sure you replace /dev/sdx with your drive and do not append the partition number. Also, replace /path/to/CentOS-7-x86_64-DVD-1810.iso with the path to the ISO file. If you downloaded the file using a web browser then it should be stored in the Downloads folder located in your user account.

```bash
sudo dd bs=4M if=/path/to/CentOS-7-x86_64-DVD-1810.iso of=/dev/sdx status=progress oflag=sync
```

The command will show a progress bar while flashing the image.

The process may take several minutes, depending on the size of the ISO file and the USB stick speed. Once completed you will see something like below:

```bash
1094+0 records in
1094+0 records out
4588568576 bytes (4.6 GB) copied, 30.523 s, 150 MB/s
```
