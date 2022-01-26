# How to Install PHP 7.x on CentOS 8

[Original source](https://www.cyberciti.biz/faq/install-php-7-x-on-centos-8-for-nginx/)

* Open the terminal app and log in to the remote CentOS 8 server
* Update CentOS 8 box, run `sudo yum update`
* Search for PHP version, run `sudo yum search php`
* Install and enable Remi’s repo for PHP 7.4, run `sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm`
* Install PHP 7.2.11 or 7.4 and FastCGI module for Nginx on CentOS 8, execute: `sudo yum install php php-fpm`
* Configure Nginx to use PHP
* Search and install additional PHP modules for graphics and database support using `sudo yum search php-`
* Enable and restart both PHP and Nginx server
* Test and verify both PHP installation

## Update the CentOS 8 box
Run the following yum command:

```bash
sudo yum update
```

Reboot the Linux system if a new kernel installed:

```bash
sudo reboot
```

## Search for PHP version
Let us find out PHP version on CentOS Enterprise Linux 8 server, execute:

```bash
sudo yum search php-
```

You may have multiple versions of PHP installed on your systems. One can verify it by merely running the following command:

```bash
sudo yum module list php
```

Sample outputs:

```bash
Last metadata expiration check: 0:25:04 ago on Mon Dec 16 13:01:39 2019.
CentOS-8 - AppStream
Name             Stream             Profiles                              Summary                          
php              7.2 [d]            common [d], devel, minimal            PHP scripting language           

Hint: [d]efault, [e]nabled, [x]disabled, [i]nstalled
```

The following example, indicates that PHP version 7.2, 7.3 and 7.4 available for installation:

```bash
sudo yum module list php
```

Sample outputs:

```bash
Last metadata expiration check: 0:00:05 ago on Mon Dec 16 13:28:05 2019.
CentOS-8 - AppStream
Name            Stream              Profiles                              Summary                          
php             7.2 [d]             common [d], devel, minimal            PHP scripting language           

Remi\'s Modular repository for Enterprise Linux 8 - x86_64
Name            Stream              Profiles                              Summary                          
php             remi-7.2            common [d], devel, minimal            PHP scripting language           
php             remi-7.3            common [d], devel, minimal            PHP scripting language           
php             remi-7.4            common [d], devel, minimal            PHP scripting language           

Hint: [d]efault, [e]nabled, [x]disabled, [i]nstalled
```

By default, PHP version 7.2 would install as indicated by the [d] flag.

## A note about enabling different versions of PHP such as 7.3 and 7.4 on CentOS 8
I strongly suggest using default PHP version 7.2 for production web apps. However, if you need PHP version 7.3 or 7.4, type the following commands to enable Remi’s repo:

```bash
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
```

## ENABLE DEFAULT VERSION
The default PHP version locked to PHP 7.2. It would be best if you ran enable command to set the desired PHP version. In other words, to enable 

PHP version 7.4, run:

```bash
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
## verify it php set to 7.4 ##
sudo yum module list php
```

For PHP version 7.3, execute:

```bash
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.3
## verify it php set to 7.3 ##
sudo yum module list php
```

## Installing PHP on CentOS 8

Now that PHP version set, it is time to install PHP 7.x on your CentOS 8 cloud server by typing the following command:

```bash
sudo yum install php php-fpm
```

If you do not want Apache (httpd) installed as dependencies, run:

```bash
sudo yum install php-fpm php-common php-cli
```

## Enable php-fpm service

Type the following systemctl command:

```bash
sudo systemctl enable php-fpm.service
```

Start the php-fpm service, run:

```bash
sudo systemctl start php-fpm.service
sudo systemctl status php-fpm.service
```

See [how to reload/start/restart PHP-fpm service](https://www.cyberciti.biz/faq/how-to-reload-restart-php7-0-fpm-service-linux-unix/) for more info:

```bash
sudo systemctl stop php-fpm.service
sudo systemctl restart php-fpm.service
```

## How to configure PHP to work with Nginx server

First, find out the location of PHP-FPM FastCGI server config using the cat command

```bash
cat /etc/nginx/conf.d/php-fpm.conf
```

Make sure Unix socket is up and running, run:

```bash
ls -l /run/php-fpm/www.sock
```

My php-fpm config for CentOS 8 with Nginx:

```bash
cat /etc/nginx/default.d/php.conf
```

```bash
index index.php index.html index.htm;

location ~ \.php$ {
   try_files $uri =404;
   fastcgi_intercept_errors on;
   fastcgi_index index.php;
   include fastcgi_params;
   fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
   fastcgi_pass php-fpm;
}
```

## Restart the nginx service/server

Again, run the systemctl command:

```bash
sudo systemctl restart nginx.service
```

Verify php version. To find PHP version, type:

```bash
php --version
```

```bash
PHP 7.4.7 (cli) (built: Jun  9 2020 10:57:17) ( NTS )
Copyright (c) The PHP Group
Zend Engine v3.4.0, Copyright (c) Zend Technologies
    with Zend OPcache v7.4.7, Copyright (c), by Zend Technologies
```

## Test and verify that PHP 7.x is working on CentOS 8 along with Nginx

Create a new file as follows:

```bash
nano /usr/share/nginx/html/hello.php
```

Append the following PHP code:

```php
<?php
	echo "Hello, world!\n";
?>
```

Run it as follows using the curl command

```bash
curl -I http://localhost/hello.php
curl http://localhost/hello.php
```


## How to install additional php modules

Try searching and installing additional modules as follows:

```bash
sudo yum search php-
sudo yum search php- | grep mysql
sudo yum search php74- ## for version 7.2 ##
sudo yum search php74- ## for version 7.3 ##
sudo yum search php74- ## for version 7.4 ##
```

## Install PHP 7.x CentOS 8 modules

For example, install grphics and database support, run:

```bash
sudo yum install php-mysqlnd php-gd
```

Typical WordPress installation on CentOS 8 needs the following PHP extensions or modules:

```bash
sudo yum install php-mysqlnd php-gd php-pecl-zip php-mbstring php-xml php-opcache php-pecl-imagick
```

## How to configure PHP 7.x

You need to edit the following files as per your needs:

* `/etc/php.ini` – PHP’s initialization and config file. Do not modify this file. Instead create custom.ini in /etc/php.d/ directory.
* `/etc/php-fpm.conf` – Gloable FPM (FastCGI) configuration file.
* `/etc/php-fpm.d/www.conf` – FastCGI (FPM) www pool config file.
* `/etc/php.d/` – PHP modules config file.

See [best PHP security practices for web apps](https://www.cyberciti.biz/tips/php-security-best-practices-tutorial.html) for more info.

## Conclusion

And, there you have it PHP installed and running on CentOS 8. By default, PHP version 7.2 installed on CentOS 8 Linux. However, devloper or sysadmin can install the latest version such as PHP 7.4 using the Remi repository for CentOS Enterprise Linux 8 server.