# Intro

## Install nano editor

``` bash
yum install nano
```

## Install Visual Studio Code

``` bash
sudo snap install --classic code # or code-insiders
```

Once installed, the Snap daemon will take care of automatically updating VS Code in the background. You will get an in-product update notification whenever a new update is available.

Note: If snap isn't available in your Linux distribution, please check the following guide, which can help you get that set up.

### RHEL, Fedora, and CentOS based distributions

We currently ship the stable 64-bit VS Code in a yum repository, the following script will install the key and repository:

``` bash
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
sudo sh -c 'echo -e "[code]\nname=Visual Studio Code\nbaseurl=https://packages.microsoft.com/yumrepos/vscode\nenabled=1\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc" > /etc/yum.repos.d/vscode.repo'
```

Then update the package cache and install the package using dnf (Fedora 22 and above):

``` bash
dnf check-update
sudo dnf install code
```

Or on older versions using yum:

``` bash
yum check-update
sudo yum install code
```

Due to the manual signing process and the system we use to publish, the yum repo may lag behind and not get the latest version of VS Code immediately.

## Google Chrome

``` bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_x86_64.rpm
```

``` bash
yum localinstall google-chrome-stable_current_x86_64.rpm
```
