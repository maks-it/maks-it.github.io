# Install Fedora on Windows Subsystem for Linux (WSL)

https://dev.to/bowmanjd/install-fedora-on-windows-subsystem-for-linux-wsl-4b26


## Enable WSL2

```powershell
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux,VirtualMachinePlatform -All
```

```poweshell
wsl --set-default-version 2
```

## Where to download Fedora rootfs 

https://koji.fedoraproject.org/koji/packageinfo?packageID=26387

https://koji.fedoraproject.org/koji/buildinfo?buildID=2208158


Find the right "xz" file for your platform (likely x86_64). Such as:


https://kojipkgs.fedoraproject.org//packages/Fedora-Container-Base/38/20230602.0/images/Fedora-Container-Base-38-20230602.0.x86_64.tar.xz


## Extract custom rootfs

Unpack the Fedora-Container-Base-*.tar.xz file in your preferred manner.

Once unpacked you will see a folder with a long hexadecimal name. Within that folder, there should be a layer.tar file. This is your rootfs. Copy the layer.tar file to a logical location, such as your Downloads folder. You may even want to rename it to something like fedora-38-rootfs.tar.

```powershell
mkdir $HOME\wsl\fedora
```

```powershell
wsl --import fedora $HOME\wsl\fedora $HOME\Downloads\fedora-38-rootfs.tar
```

View installed distros:

```poweshell
PS C:\Users\me> wsl -l
Windows Subsystem for Linux Distributions:
fedora (Default)
```

If you have multiple distros installed, and want Fedora to be set as the default, something like this should work:

```powershell
wsl -s fedora
```

Launch Fedora as root:

```powershell
wsl -d fedora
```


## In case mnt not working


```bash
dnf install -y util-linux
```

Then exit, and terminate your fedora instance (this, in effect, causes a restart):

```bash
wsl -t fedora
```


## Set non root user

Launch Fedora as an unprivileged user

```bash
dnf install -y passwd cracklib-dicts
```

```bash
useradd -G wheel myusername
```

```bash
passwd myusername
```

Now, exit WSL or launch a new Powershell window, then re-launch WSL with the new username:

```bash
wsl -d fedora -u myusername
```

Success?

```bash
$ whoami
myusername
```

Does sudo work?

```bash
sudo cat /etc/shadow
```

If you see the list of users, including, toward the bottom, the one you recently added, then all is well!

Set the default user


```bash
printf "\n[user]\ndefault = myusername\n" | sudo tee -a /etc/wsl.conf
```

```powershell
wsl -t fedora
```

Launch WSL again, without specifying a user, and you should be that user, not root.


## Fine tuning


If you do container work, especially in userspace, you will likely want to reinstall shadow-utils, in order to fix sticky bits that weren't set properly in the rootfs:


```bash
sudo dnf reinstall -y shadow-utils
```


If you like to ping servers to see if they are up, then these two steps may be necessary:

```bash
sudo dnf install -y procps-ng iputils
sudo sysctl -w net.ipv4.ping_group_range="0 2000"

```

The second one allows group IDs all the through 2000 to be able to ping. You can check group IDs with getent group or see your primary group ID with id -g and make sure it is included in the range above.

To make the above permanent, however, it is necessary to create or alter the $HOME\.wslconfig file in Windows, not Linux. Placing the following in that file will allow ping to work even after restarts:

```
[wsl2]
kernelCommandLine = sysctl.net.ipv4.ping_group_range=\"0 2000\"
```


```bash
sudo dnf -y install iproute findutils ncurses
````

```bash
sudo dnf install nano -y
```

## A few good man pages

First, ensure the nodocs option is not set in /etc/dnf/dnf.conf. You may edit out the tsflags=nodocs line yourself, or use the following:

```bash
grep -v nodocs /etc/dnf/dnf.conf | sudo tee /etc/dnf/dnf.conf

```


```bash
# see `man dnf.conf` for defaults and possible options

[main]
gpgcheck=True
installonly_limit=3
clean_requirements_on_remove=True
best=False
skip_if_unavailable=True
```

Then install man and man-pages:

```bash
sudo dnf install -y man man-pages
```

This will ensure you get man pages on every future dnf install; however, to add them in retroactively, you will want to dnf reinstall any package for which you want man pages. For instance, man dnf will yield nothing now. But try it again after sudo dnf reinstall -y dnf and you should have good results.

To reinstall all installed packages, try the following:

```bash
for pkg in $(dnf repoquery --installed --qf "%{name}"); do sudo dnf reinstall -qy $pkg; done
```


## Export custom tarball

```bash
sudo dnf clean all
```

```bash
wsl --export fedora $HOME\Downloads\fedora-wsl.tar
```





