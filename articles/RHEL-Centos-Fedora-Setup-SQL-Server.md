# SQL Server on Linux

SQL Server runs on Linux starting with SQL Server 2017. This SQL Server is the same SQL Server database engine running on Microsoft Operating systems, with many similar features and services.
This guide will take you through the steps to install Microsoft SQL Server 2019 on CentOS 7 | Fedora 32/31/30/29/ Linux system. As of this writing, SQL Server 2019 is available for Production use on Ubuntu / CentOS and RHEL Linux.

Step 1: Install Microsoft SQL Server 2019 on CentOS 7 | Fedora 32/31/29/28
Microsoft SQL Server 2019 is available for the general use. Add the repository to your CentOS 7 / Fedora by running the following commands on your terminal.

sudo curl -o /etc/yum.repos.d/mssql-server.repo https://packages.microsoft.com/config/rhel/8/mssql-server-2019.repo
This will download  SQL Server 2019 repository to /etc/yum.repos.d/mssql-server.repo

Update your system cache:

```bash
sudo yum makecache  # CentOS 8
sudo dnf makecache  # Fedora
```

Then install SQL server 2019:

```bash
sudo yum install -y mssql-server
```

For Fedora, run:

``bash
sudo dnf install -y mssql-server
To get info about the installed package, run:
$ rpm -qi mssql-server
Name        : mssql-server
Version     : 15.0.1100.94
Release     : 1
Architecture: x86_64
Install Date: Sat 17 Nov 2018 09:12:15 AM UTC
Group       : Unspecified
Size        : 1289243002
License     : Commercial
Signature   : RSA/SHA256, Tue 06 Nov 2018 10:12:05 PM UTC, Key ID eb3e94adbe1229cf
Source RPM  : mssql-server-15.0.1100.94-1.src.rpm
Build Date  : Tue 06 Nov 2018 08:47:29 AM UTC
Build Host  : hls-cent3-prod-build-cent73-03
Relocations : (not relocatable)
Summary     : Microsoft SQL Server Relational Database Engine
Description :
The mssql-server package contains the Microsoft SQL Server Relational Database Engine.
```

Step 2: Initialize MS SQL Database Engine
After the package installation finishes, run mssql-conf setup and follow the prompts to set the SA password and choose your edition.

```bash
sudo /opt/mssql/bin/mssql-conf setup
```

1. Select an edition you’d like to use

```bash
Choose an edition of SQL Server:
  1) Evaluation (free, no production use rights, 180-day limit)
  2) Developer (free, no production use rights)
  3) Express (free)
  4) Web (PAID)
  5) Standard (PAID)
  6) Enterprise (PAID)
  7) Enterprise Core (PAID)
  8) I bought a license through a retail sales channel and have a product key to enter.
```

For me. I’ll go with 2 – Developer (free, no production use rights).

2. Accept the license terms

```bash
The license terms for this product can be found in
/usr/share/doc/mssql-server or downloaded from:
https://go.microsoft.com/fwlink/?LinkId=855862&clcid=0x409

The privacy statement can be viewed at:
https://go.microsoft.com/fwlink/?LinkId=853010&clcid=0x409

Do you accept the license terms? [Yes/No]:Yes
```

3. Set SQL Server system administrator password

```bash
Enter the SQL Server system administrator password: <Password>
Confirm the SQL Server system administrator password:<Confirm Password>
Configuring SQL Server...

sqlservr: This program requires a machine with at least 2000 megabytes of memory.
/opt/mssql/bin/sqlservr: This program requires a machine with at least 2000 megabytes of memory.

Initial setup of Microsoft SQL Server failed. Please consult the ERRORLOG
in /var/opt/mssql/log for more information.
```

Step 3: Install SQL Server command-line tools
Then install mssql-tools with the unixODBC developer package.

```bash
sudo curl -o /etc/yum.repos.d/mssql-tools.repo https://packages.microsoft.com/config/rhel/8/prod.repo
sudo yum -y install mssql-tools unixODBC-devel
```

Step 4: Start and enable mssql-server  service

Start mssql-server  service

```bash
sudo systemctl start mssql-server
```

Enable it to start on system boot:

```bash
sudo systemctl enable mssql-server
```

Add /opt/mssql/bin/ to your $PATH variable:

```bash
echo 'export PATH=$PATH:/opt/mssql/bin:/opt/mssql-tools/bin' | sudo tee /etc/profile.d/mssql.sh
```

Source the file to start using MS SQL executable binaries in your current shell session:

```bash
source /etc/profile.d/mssql.sh
```

If you have an active Firewalld service, allow SQL Server ports for remote hosts to connect:

```bash
sudo  firewall-cmd --add-port=1433/tcp --permanent
sudo  firewall-cmd --reload
```

Step 4: Test SQL Server
Connect to the SQL Server and verify it is working.

```bash
$ sqlcmd -S localhost -U SA
```
Authenticate with the password set in Step 2.

Show Database users:
```bash
1> select name from sysusers;
2> go
```

Create a test database:

Create new:

```bahs
CREATE DATABASE mytestDB
SELECT Name from sys.Databases
GO
USE mytestDB
CREATE TABLE Inventory (id INT, name NVARCHAR(50), quantity INT)
INSERT INTO Inventory VALUES (1, 'banana', 150); INSERT INTO Inventory VALUES (2, 'orange', 154);
GO
SELECT * FROM Inventory LIMIT 1;
Show databases on the SQL Server.

1> select name,database_id from sys.databases;
2> go
```

Drop a database:

```bash
1> drop database mytestDB;
2> go
```


To configure the remote access option Maybe not necessary

Connect to the Database Engine.

From the Standard bar, click New Query.

Copy and paste the following example into the query window and click Execute. This example shows how to use sp_configure to set the value of the remote access option to 0.

```SQL
EXEC sp_configure 'remote access', 0 ;  
GO  
RECONFIGURE ;  
GO  
```

Restart the server

```bash
sudo systemctl restart mssql-server
```

## Reset SA password

Connect SQL Server using command-line tool with the existing password (The purpose of this step to show you my current “sa” password. At the end, once I reset the “sa” password, you will see that I shall be using a new “sa” password to connect to SQL Server. If you are dealing with real time issue, you can ignore this step.)

```bash
sqlcmd -S <SQLInstanceName>-U <UserName> -P <Password>
```

To change the “sa” password, you have to first Stop SQL Server service which is running on the Linux machine. Let’s stop the SQL Server and verify the status of SQL Server.

```bash
sudo systemctl stop mssql-server
sudo systemctl status mssql-server
```

Reset the “sa” password by key new strong password now.

```bash
/opt/mssql/bin/mssql-conf set-sa-password
```

>Note: When you are resetting/changing “sa” password using sqlcmd in a bash terminal,  you need to be escaped or not used the character “$” because it is a special character in bash.

Start and verify the status of SQL Server Service

```bash
sudo systemctl start mssql-server
sudo systemctl status mssql-server
```

Let’s connect SQL Server with the new password.

```bash
sqlcmd -S <SQLInstanceName>-U <UserName> -P <Password>
```
