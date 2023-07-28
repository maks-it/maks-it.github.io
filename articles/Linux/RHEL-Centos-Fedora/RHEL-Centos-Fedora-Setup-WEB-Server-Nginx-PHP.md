# How to Install PHP with Nginx

## Install PHP

```bash 
sudo dnf install php-fpm php-cli php-mysqlnd php-dom php-imagick php-mbstring php-zip php-gd php-intl -y
```

```bash
 systemctl start php-fpm --now
```

```bash
 php-fpm -v
```

```bash
nano /etc/php-fpm.d/www.conf
```

```bash
listen.owner = nginx
listen.group = nginx
listen.mode = 0660

```

## Install Nginx

```bash
sudo dnf install nginx
```

```bash
sudo systemctl enable nginx --now
```

```bash
sudo mkdir /etc/nginx/sites-available
sudo mkdir /etc/nginx/sites-enabled
```

>>>
**Note**: This directory layout was introduced by Debian contributors, but we are including it here for added flexibility with managing our server blocks (as it's easier to temporarily enable and disable server blocks this way).
>>>

Next, we should tell Nginx to look for server blocks in the `sites-enabled` directory. To accomplish this, we will edit Nginx's main configuration file and add a line declaring an optional directory for additional configuration files:

``` bash
sudo nano /etc/nginx/nginx.conf
```

Add these lines to the end of the `http {}` block:

``` bash
### Look for server blocks in the sites-enabled directory ###
include "/etc/nginx/sites-enabled/*.conf";
server_names_hash_bucket_size 64;


client_max_body_size 100M;
```

The first line instructs Nginx to look for server blocks in the `sites-enabled` directory, while the second line increases the amount of memory that is allocated to parsing domain names (since we are now using multiple domains).


Now, open the new file in your text editor with root privileges:

``` bash
sudo nano /etc/nginx/sites-available/\<mysite\>.conf
```

[NGINX PHP-FPN](https://www.nginx.com/resources/wiki/start/topics/examples/phpfcgi/)

If you’re using unix socket change fastcgi_pass to:

```bash
fastcgi_pass unix:/var/run/php-fpm/www.sock;
```

``` bash
server {
    listen       80;
    server_name  \<mysite\> www.\<mysite\>;
    root         /var/www/html/\<mysite\>;

    access_log /var/log/nginx/\<mysite\>-access.log;
    error_log  /var/log/nginx/\<mysite\>-error.log error;
    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ /index.php$is_args$args;
    }

    location ~ [^/]\.php(/|$) {
        fastcgi_split_path_info ^(.+?\.php)(/.*)$;
        if (!-f $document_root$fastcgi_script_name) {
            return 404;
        }

        # Mitigate https://httpoxy.org/ vulnerabilities
        fastcgi_param HTTP_PROXY "";

        # fastcgi_pass 127.0.0.1:9000;
        fastcgi_pass unix:/var/run/php-fpm/www.sock;
        fastcgi_index index.php;

        # include the fastcgi_param setting
        include fastcgi_params;

        # SCRIPT_FILENAME parameter is used for PHP FPM determining
        #  the script name. If it is not set in fastcgi_params file,
        # i.e. /etc/nginx/fastcgi_params or in the parent contexts,
        # please comment off following line:
        fastcgi_param  SCRIPT_FILENAME   $document_root$fastcgi_script_name;
    }
}
```

## Enable site

```bash
sudo ln -s /etc/nginx/sites-available/\<mysite\>.conf /etc/nginx/sites-enabled/\<mysite\>.conf
```

## Install mariadb

```bash
dnf install mariadb-server -y
```

```bash
systemctl enable mariadb --now
```

```bash
mysql -u root -p
SELECT version();
```

```sql
CREATE USER 'your_username'@'host_ip_addr' IDENTIFIED BY 'your_password';

GRANT ALL PRIVILEGES ON *.* TO 'your_username'@'%'
  IDENTIFIED BY 'my-new-password' WITH GRANT OPTION;
```


## Open firewall ports



## Install wordpress

utf8mb4_general_ci

```bash
chown nginx:nginx  -R * # Let NGINX be owner

```

```bash
chmod -R 777 /var/www/html/<sitename>
```

enable direct method without ftp

define( 'FS_METHOD', 'direct' );


