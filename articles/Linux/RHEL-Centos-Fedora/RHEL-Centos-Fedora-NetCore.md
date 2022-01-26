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

```bash
$ sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
$ sudo wget -O /etc/yum.repos.d/microsoft-prod.repo https://packages.microsoft.com/config/fedora/33/prod.repo
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

