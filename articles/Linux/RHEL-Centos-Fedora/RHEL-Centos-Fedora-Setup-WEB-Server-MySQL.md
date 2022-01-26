
## Step 1 – Setup Yum Repository

In the first step install all the required yum repositories in your system used in the remaining tutorial for various installations. You are adding REMI, EPEL, Webtatic & MySQL community server repositories in your system.
CentOS / RHEL 7

``` bash
yum install epel-release
rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
rpm -Uvh http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
```

CentOS / RHEL 6

``` bash
yum install epel-release
rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-6.rpm
rpm -Uvh http://repo.mysql.com/mysql-community-release-el6-5.noarch.rpm
```

## Step 4 – Install MySQL 5.6

In step 1 we already have installed required yum repository in your system. Lets use following command to install MySQL server on your system.

``` bash
yum install mysql-server
```

You need to execute mysql_secure_installation once after installation of MySQL server using following command. First it will prompt to set a password for root account, after that ask few questions, I suggest to say yes ( y ) for all.

``` bash
systemctl start mysqld.service
mysql_secure_installation
```

Now restart MySQL service and enable to start on system boot.

``` bash
systemctl restart mysqld.service
systemctl enable mysqld.service
```