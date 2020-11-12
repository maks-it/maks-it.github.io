# Install postgresql

Follow guide on postgresql download page

For RHEL / CentOS / SL / OL 7, 8 Or Fedora 29 And Later Derived Distributions:

```bash
  postgresql-setup initdb
  systemctl enable postgresql.service
  systemctl start postgresql.service
```

## Configure password

PostgreSQL Basic Setup
In Linux by default, a user named postgres is created once PostgreSQL is installed. You can change the user's password with the following command:

```bash
sudo passwd postgres
```

You will be prompted to enter the new password twice.

Next, you can switch to the PostgreSQL prompt and change the password for the PostgreSQL postgres user using:

```bash
su - postgres
```

If you receive an error, you can set a valid shell on the user with the following command:

```bash
su --shell /bin/bash postgres
```

Afterwards, perform the same command:

```bash
su - postgres
```

To change the password, use the below command where you add your new password instead of the NewPassword:

```bash
psql -d template1 -c "ALTER USER postgres WITH PASSWORD 'NewPassword';"
```

You can switch to the PostgreSQL client shell using:

```bash
psql postgres
```

Here you can check the list of available commands by typing \h. You can use \h followed by the command for which you need more information. To exit the environment you can type \q.

The createdb command lets you create new databases. Suppose we want to create a new database named testDB using the postgres Linux user. The command we would use would look like this:

```bash
createdb testDB
```

You can create a new role using the createuser command. Below is an example where we are creating a role named samplerole using the postgres Linux user.

```bash
createuser samplerole –pwprompt
```

Here you will be prompted to set a password for the user.

Optionally you can assign the ownership of our newly created database to a specific postgres user or role. This can be done with a command like this one:

```bash
createdb testDB -O samplerole
```

In the above command, replace samplerole with the role you want to use.

You can connect to this new database using the command bellow:

```bash
psql testDB
```

In case you want to use a specific user or role to log in, use the command as shown below:

```bash
psql testDB -U samplerole
```

This will prompt you to enter the password.

You can use \l or \list commands to show all the databases. To know the current database you’re using, you can use \c. In case you want more information about connections such as the socket, port, etc.  then you can use \conninfo.

You can also drop or delete a database using the dropdb command. However, remember to verify what you’re deleting before doing it. Deleted databases cannot be retrieved.

To delete a database, you can use:

```bash
dropdb testDB
```

## Allow remote access

By default PostgreSQL uses IDENT-based authentication and this will never allow you to login via -U and -W options. Allow username and password based authentication from your application by appling 'trust' as the authentication method for the Bitbucket database user. You can do this by modifying the pg_hba.conf file.

You can identify the location of the pg_hba.conf file by running the following command in psql command line, you'll need to be logged in as a superuser in the database:

```bash
postgres=# show hba_file ;
 hba_file
--------------------------------------
 /etc/postgresql/9.3/main/pg_hba.conf
(1 row)
```

set for example:

```bash
local    all    all    trust
host    all    127.0.0.1/32    trust
```

Don't forget to restart PostgreSQL after saving your changes to the file.

```bash
# service postgresql restart
```

## Exit psql user

```bash
    \q
```


## How To backup and restore

Login as postgres user

```bash
su - postgres
```

backup
```bash
pg_dump name_of_database > name_of_backup_file
```

restore
```bash
psql empty_database < backup_file
```
