# Intro

## Publish .Net Core application

Run dotnet publish from the development environment to package an app into a
directory (for example, bin/Release/<target_framework_moniker>/publish) that can run
on the server:

``` bash
dotnet publish --configuration Release
```

## Configure NGINX reverse proxy server

A reverse proxy is a common setup for serving dynamic web apps. A reverse proxy
terminates the HTTP/HTTPS request and forwards it to the ASP.NET Core app.

### Setup Proxy

Add the proxy configuration file

``` bash
/etc/nginx/proxy.conf
```

and include it into http section of

```bash
/etc/nginx/nginx.conf
```

### Setup Server block for particular site

To configure Nginx as a reverse proxy to forward requests to your ASP.NET Core app,
modify /etc/nginx/sites-available/example.com.conf

## Create .Net Core app Service

Create the service definition file:

bash ```
sudo nano /etc/systemd/system/kestrel-example.com.service
```

Enable service

``` bash
sudo systemctl enable kestrel-example.com.service
```

Start service

``` bash
sudo systemctl start kestrel-example.com.service
```

Get dervice status
``` bash
sudo systemctl status kestrel-example.com.service -l
```

## View logs

Since the web app using Kestrel is managed using systemd , all events and processes are
logged to a centralized journal. However, this journal includes all entries for all services
and processes managed by systemd . To view the kestrel-helloapp.service -specific
items, use the following command:

``` bash
sudo journalctl -fu kestrel-helloapp.service
```

For further filtering, time options such as --since today , --until 1 hour ago or a
combination of these can reduce the amount of entries returned.

``` bash
sudo journalctl -fu kestrel-helloapp.service --since "2016-10-18" --until
"2016-10-18 04:00"
```

## In case omnisharp is not able to find .net core

* Remove every package listed by
```
dnf history userinstalled | grep dotnet
```
* Delete dotnet folders
```
sudo rm -rf /usr/share/dotnet
sudo rm -rf /usr/lib64/dotnet
```
* Remove microsoft repository
```
sudo rm -rf /etc/yum.repos.d/microsoft-prod.repo
```
* Reinstall dotnet sdk
```
sudo dnf -y install dotnet-sdk-3.1
```
