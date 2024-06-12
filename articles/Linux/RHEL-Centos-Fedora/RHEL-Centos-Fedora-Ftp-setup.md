To configure `vsftpd` (Very Secure FTP Daemon) to enable a read-only folder for anonymous users, you need to make some specific changes to the `vsftpd` configuration file, usually located at `/etc/vsftpd/vsftpd.conf`. Here's how you can do it on an AlmaLinux system:

1. **Install vsftpd if it's not already installed**:
```bash
sudo dnf install -y vsftpd
```

1. **Open the configuration file**:
```bash
sudo nano /etc/vsftpd/vsftpd.conf
```

1. **Modify or ensure these settings are in the configuration file**:
- `anonymous_enable=YES`: Allows anonymous FTP access.
- `local_enable=NO`: Disables login from local users if you only want anonymous access.
- `write_enable=NO`: Prevents all users from writing to the filesystem, ensuring read-only access.
- `anon_root=/var/ftp/pub`: Specifies the directory anonymous users will access. Adjust the path as needed for your specific setup.
- `anon_upload_enable=NO`: Ensures that files cannot be uploaded.
- `anon_mkdir_write_enable=NO`: Prevents the creation of new directories.

Here's how your `vsftpd.conf` might look:

```plaintext
anonymous_enable=YES
local_enable=NO
write_enable=NO
anon_root=/var/ftp/pub
anon_upload_enable=NO
anon_mkdir_write_enable=NO
no_anon_password=YES
hide_ids=YES
```

1. **Set up the directory for anonymous access**:
You might need to adjust permissions and ownership to make sure that the directory is accessible to the `ftp` user.

```bash
sudo mkdir -p /var/ftp/pub
sudo chown ftp:ftp /var/ftp/pub
sudo chmod 755 /var/ftp/pub
```

to allow wheel users to write there via ssh

```bash
sudo mkdir -p /var/ftp/pub
sudo chown ftp:wheel /var/ftp/pub
sudo chmod 775 /var/ftp/pub
```

5. **Restart vsftpd to apply the changes**:
```bash
sudo systemctl restart vsftpd
```

6. **Enable vsftpd to start on boot**:
```bash
sudo systemctl enable vsftpd
```

7. **Configure your firewall**:
If you're using `firewalld`, you might need to allow FTP traffic:
```bash
sudo firewall-cmd --zone=public --add-service=ftp --permanent
sudo firewall-cmd --reload
```

This setup will provide a basic anonymous FTP server where users can only download files from the `/var/ftp/pub` directory and cannot make any changes. Make sure to further secure your FTP server as needed, depending on your specific requirements and environment.