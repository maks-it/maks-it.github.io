# RabbitMQ setup

[https://www.rabbitmq.com/install-rpm.html](https://www.rabbitmq.com/install-rpm.html)

Create file

```bash
sudo nano /etc/yum.repos.d/rabbitmq.repo
```

and paste following content

```bash
##
## Zero dependency Erlang RPM
##

[modern-erlang]
name=modern-erlang-el9
# uses a Cloudsmith mirror @ yum.novemberain.com.
# Unlike Cloudsmith, it does not have any traffic quotas
baseurl=https://yum1.novemberain.com/erlang/el/9/$basearch
        https://yum2.novemberain.com/erlang/el/9/$basearch
        https://dl.cloudsmith.io/public/rabbitmq/rabbitmq-erlang/rpm/el/9/$basearch
repo_gpgcheck=1
enabled=1
gpgkey=https://github.com/rabbitmq/signing-keys/releases/download/3.0/cloudsmith.rabbitmq-erlang.E495BB49CC4BBE5B.key
gpgcheck=1
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300
pkg_gpgcheck=1
autorefresh=1
type=rpm-md

[modern-erlang-noarch]
name=modern-erlang-el9-noarch
# uses a Cloudsmith mirror @ yum.novemberain.com.
# Unlike Cloudsmith, it does not have any traffic quotas
baseurl=https://yum1.novemberain.com/erlang/el/9/noarch
        https://yum2.novemberain.com/erlang/el/9/noarch
        https://dl.cloudsmith.io/public/rabbitmq/rabbitmq-erlang/rpm/el/9/noarch
repo_gpgcheck=1
enabled=1
gpgkey=https://github.com/rabbitmq/signing-keys/releases/download/3.0/cloudsmith.rabbitmq-erlang.E495BB49CC4BBE5B.key
       https://github.com/rabbitmq/signing-keys/releases/download/3.0/rabbitmq-release-signing-key.asc
gpgcheck=1
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300
pkg_gpgcheck=1
autorefresh=1
type=rpm-md

[modern-erlang-source]
name=modern-erlang-el9-source
# uses a Cloudsmith mirror @ yum.novemberain.com.
# Unlike Cloudsmith, it does not have any traffic quotas
baseurl=https://yum1.novemberain.com/erlang/el/9/SRPMS
        https://yum2.novemberain.com/erlang/el/9/SRPMS
        https://dl.cloudsmith.io/public/rabbitmq/rabbitmq-erlang/rpm/el/9/SRPMS
repo_gpgcheck=1
enabled=1
gpgkey=https://github.com/rabbitmq/signing-keys/releases/download/3.0/cloudsmith.rabbitmq-erlang.E495BB49CC4BBE5B.key
       https://github.com/rabbitmq/signing-keys/releases/download/3.0/rabbitmq-release-signing-key.asc
gpgcheck=1
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300
pkg_gpgcheck=1
autorefresh=1


##
## RabbitMQ Server
##

[rabbitmq-el9]
name=rabbitmq-el9
baseurl=https://yum2.novemberain.com/rabbitmq/el/9/$basearch
        https://yum1.novemberain.com/rabbitmq/el/9/$basearch
        https://dl.cloudsmith.io/public/rabbitmq/rabbitmq-server/rpm/el/9/$basearch
repo_gpgcheck=1
enabled=1
# Cloudsmith's repository key and RabbitMQ package signing key
gpgkey=https://github.com/rabbitmq/signing-keys/releases/download/3.0/cloudsmith.rabbitmq-server.9F4587F226208342.key
       https://github.com/rabbitmq/signing-keys/releases/download/3.0/rabbitmq-release-signing-key.asc
gpgcheck=1
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300
pkg_gpgcheck=1
autorefresh=1
type=rpm-md

[rabbitmq-el9-noarch]
name=rabbitmq-el9-noarch
baseurl=https://yum2.novemberain.com/rabbitmq/el/9/noarch
        https://yum1.novemberain.com/rabbitmq/el/9/noarch
        https://dl.cloudsmith.io/public/rabbitmq/rabbitmq-server/rpm/el/9/noarch
repo_gpgcheck=1
enabled=1
# Cloudsmith's repository key and RabbitMQ package signing key
gpgkey=https://github.com/rabbitmq/signing-keys/releases/download/3.0/cloudsmith.rabbitmq-server.9F4587F226208342.key
       https://github.com/rabbitmq/signing-keys/releases/download/3.0/rabbitmq-release-signing-key.asc
gpgcheck=1
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300
pkg_gpgcheck=1
autorefresh=1
type=rpm-md

[rabbitmq-el9-source]
name=rabbitmq-el9-source
baseurl=https://yum2.novemberain.com/rabbitmq/el/9/SRPMS
        https://yum1.novemberain.com/rabbitmq/el/9/SRPMS
        https://dl.cloudsmith.io/public/rabbitmq/rabbitmq-server/rpm/el/9/SRPMS
repo_gpgcheck=1
enabled=1
gpgkey=https://github.com/rabbitmq/signing-keys/releases/download/3.0/cloudsmith.rabbitmq-server.9F4587F226208342.key
gpgcheck=0
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300
pkg_gpgcheck=1
autorefresh=1
type=rpm-md
```

```bash
dnf update -y
```

Next install dependencies from the standard repositories:

```bash
## install these dependencies from standard OS repositories
dnf install socat logrotate -y

```

Then, install modern Erlang and RabbitMQ:

```bash
## install RabbitMQ and zero dependency Erlang
dnf install -y erlang rabbitmq-server
```

Enable Web based user interface

```bash
rabbitmq-plugins enable rabbitmq_management
```

## Open firewall ports

http://{node-hostname}:15672

Create service file

```bash
sudo nano /etc/firewalld/services/rabbitmq.xml
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<service>
  <port port="5672" protocol="tcp"/>
  <port port="15672" protocol="tcp"/>
</service>
```

```bash
sudo firewall-cmd --add-service=rabbitmq --permanent
```

## Start the Server

```bash
systemctl start --now rabbitmq-server
```

## Create admin account

```bash
rabbitmqctl add_user \<username\> \<password\>
```

```bash
rabbitmqctl set_user_tags maksym administrator
```

## Setting up NGINX reverse proxy

```bash
dnf install nginx-mod-stream
```

Allow AMQP protocol proxy

```bash
nano /etc/nginx/nginx.conf
```

```bash
stream {
    upstream rabbitmq {
       server \<server address or ip\>:\<port\>;
    }

    server {
      listen 5672;
      proxy_pass rabbitmq;
      proxy_connect_timeout 1s;
      proxy_timeout 3s;
   }
}
```

```bash

server {
  listen *:443 http2 ssl;

  server_name \<host name or ip\>;

  access_log /var/log/nginx/\<host name\>-access.log;
  error_log  /var/log/nginx/\<host name\>-error.log error;

  ssl_certificate /var/www/ssl/maks-it.com/maks-it.com.crt;
  ssl_certificate_key /var/www/ssl/maks-it.com/maks-it.com.key;
  ssl_protocols             TLSv1.1 TLSv1.2;
  ssl_prefer_server_ciphers on;
  ssl_ciphers               "EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH";
  ssl_ecdh_curve            secp384r1;
  ssl_session_cache         shared:SSL:10m;
  ssl_session_tickets       off;
  ssl_stapling              on; #ensure your cert is capable
  ssl_stapling_verify       on; #ensure your cert is capable

  # Redirect all traffic
  location / {
    proxy_pass \<destination host name\>:15672;

    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }
}

```