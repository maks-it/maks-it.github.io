# .Net Core

``` bash
sudo rpm -Uvh https://packages.microsoft.com/config/rhel/7/packages-microsoft-prod.rpm
```

Update the products available for installation, then install the .NET SDK.

In your terminal, run the following commands:

``` bash
sudo yum update
sudo yum install dotnet-sdk-2.2
```

## Create React app

``` bash
dotnet new react -o my-new-app
cd my-new-app
```

## Create Console app

``` bash
dotnet new console -o my-new-app
cd my-new-app
```

## System limit on watched files by a user

The system has a limit to how many files can be watched by a user. You can run out of watches pretty quickly if you have Grunt running with other programs like Dropbox. This command increases the maximum amount of watches a user can have.

```  bash
echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf && sudo sysctl -p
```

For Arch Linux add this line to /etc/sysctl.d/99-sysctl.conf:

``` bash
fs.inotify.max_user_watches=524288
```

>>>
Explanation: echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf
writes at the end of the file /etc/sysctl.conf the line "fs.inotify.max_user_watches=524288"

sudo sysctl -p reconfigures the kernel at runtime, loading the file /etc/sysctl.conf as a parameter
>>>

For Arch Linux add fs.inotify.max_user_watches=524288 to /etc/sysctl.d/99-sysctl.conf and then execute sysctl --system. This will also persist across reboots. For more details: wiki.archlinux.org/index.php/Sysctl

npm dedupe cleared it up for me.

>>>
See also [this link](https://code.visualstudio.com/docs/setup/linux#_visual-studio-code-is-unable-to-watch-for-file-changes-in-this-large-workspace-error-enospc)
>>>