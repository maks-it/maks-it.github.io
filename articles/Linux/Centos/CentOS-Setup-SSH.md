# Cenros SSH Setup

## Installing and enabling

```bash
    sudo dnf install openssh-server openssh-clients
```

Starting SSH Service

```bash
    sudo systemctl start sshd
```

## OpenSSH Server Congigs

```bash
    sudo nano /etc/ssh/sshd_config
```

To disable root login:

``` bash
    PermitRootLogin no
```

Change the SSH port to run on a non-standard port. For example:

```bash
   Port 2002
```

## Selinux configuration

```bash
    semanage port -a -t ssh_port_t -p tcp 2002
```

## Firewall configuration

``` bash
    sudo firewall-cmd --permanent --zone=public --add-port=2222/tcp
    sudo firewall-cmd --reload
```

## Restrt service

```bash
    sudo systemctl restart sshd
```

## Optional

It is also possible to restrict IP access to make the connection even more secure.

```bash
   sudo nano /etc/sysconfig/iptables
```

To allow access using the port defined in the sshd config file, add following line tothe iptables file:

```bash
   -A RH-Firewall-1-INPUT -m state --state NEW -m tcp -p tcp --dport 2002 -j ACCEPT
```

To restrict access to a specific IP, for example 133.123.40.166, edit the line as follows:

```bash
   -A RH-Firewall-1-INPUT -s 133.123.40.166 -m state --state NEW -p tcp --dport 2002 -j ACCE
PT
```

If your site uses IPv6, and you are editing ip6tables, use the line:

```bash
   -A RH-Firewall-1-INPUT -m tcp -p tcp --dport 2002 -j ACCEPT
```

Restart iptables to apply the changes:

```bash
    sudo systemctl restart iptables
```


## Add server key file

Open Terminal.

Paste the text below, substituting in your GitHub email address.

```bash
$ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

This creates a new ssh key, using the provided email as a label.

```bash
> Generating public/private rsa key pair.
```

When you're prompted to "Enter a file in which to save the key," press Enter. This accepts the default file location.

```bash
> Enter a file in which to save the key (/home/you/.ssh/id_rsa): [Press enter]
```

At the prompt, type a secure passphrase. For more information, see "Working with SSH key passphrases".

```bash
> Enter passphrase (empty for no passphrase): [Type a passphrase]
> Enter same passphrase again: [Type passphrase again]
```

### Step 2: Copy Public Key to CentOS Server
You can copy the public SSH key on the remote server using several different methods:

using the ssh-copy-id script
using Secure Copy (scp)
manually copying the key
The fastest and easiest method is by utilizing ssh-copy-id. If the option is available, we recommend using it. Otherwise, try any of the other two noted.

Copy Public Key Using ssh-copy-id
1. Start by typing the following command, specifying the SSH user account, and the IP address of the remote host:

ssh-copy-id username@remote_host
If it is the first time your local computer is accessing this specific remote server you will receive the following output:

The authenticity of host '104.0.316.1 (104.0.316.1)' can't be established.
ECDSA key fingerprint is KYg355:gKotTeU5NQ-5m296q55Ji57F8iO6c0K6GUr5:PO1iRk.
Are you sure you want to continue connecting (yes/no)? yes
2. Confirm the connection – type yes and hit Enter.

3. Once it locates the id_rsa.pub key created on the local machine, it will ask you to provide the password for the remote account. Type in the password and hit Enter.

4. Once the connection has been established, it adds the public key on the remote server. This is done by copying the ~/.ssh/id_rsa.pub file to the remote server’s ~/.ssh directory. You can locate it under the name authorized_keys.

5. Lastly, the output tells you the number of keys added, along with clear instructions on what to do next:

Number of key(s) added: 1
Now try logging into the machine, with: "ssh 'username@104.0.316.1'"
and check to make sure that only the key(s) you wanted were added.
Copy Public Key Using Secure Copy
1. First, set up an SSH connection with the remote user:

ssh username@remote_host
2. Next, create the ~/.ssh directory as well as the authorized_keys file:

mkdir -p ~/.ssh && touch ~/.ssh/authorized_keys
3. Use the chmod command to change the file permission:

chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys
chmod 700 makes the file executable, while chmod 600 allows the user to read and write the file.

4. Now, open a new terminal session, on the local computer.

5. Copy the content from id_rsa.pub (the SSH public key) to the previously created authorized_keys file on the remote CentOS server by typing the command:

scp ~/.ssh/id_rsa.pub username@remote_host:~/.ssh/authorized_keys
With this, the public key has been safely stored on the remote account.

Copy Public Key Manually 
1. To manually add the public SSH key to the remote machine, you first need to open the content from the ~/.ssh/id_rsa.pub file:

cat ~/.ssh/id_rsa.pub
2. As in the image below, the key starts with ssh-rsa and ends with the username of the local computer and hostname of the remote machine:

example of a ssh public key on local machine username
3. Copy the content of the file, as you will need later.

4. Then, in the terminal window, connect to the remote server on which you wish to copy the public key. Use the following command to establish the connection:

ssh username@remote_host
5. Create a ~/.ssh directory and authorized_keys file on the CentOS server with the following command:

mkdir -p ~/.ssh && touch ~/.ssh/authorized_keys
6. Change their file permission by typing:

chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys
7. Next, open the authorized_keys file with an editor of your preference. For example, to open it with Nano, type:

nano authorized_keys
8. Add the public key, previously copied in step 2 of this section, in a new line in (under the existing content).

9. Save the changes and close the file.

10. Finally, log into the server to verify that everything is set up correctly.

### Step 3: Connect to Remote Server Using SSH Keys
Once you have completed the previous steps (creating an RSA Key Pair and copying the Public Key to the CentOS server), you will be able to connect to the remote host without typing the password for the remote account.

All you need to do is type in the following command:

ssh username@remote_host
If you didn’t specify a passphrase while creating the SSH key pair, you will automatically log in the remote server.

Otherwise, type in the passphrase you supplied in the initial steps and press Enter.

Once the shell confirms the key match, it will open a new session for direct communication with the server.

### Step 4: Disable Password Authentication

1. Using the SSH keys, log into the remote CentOS server which has administrative privileges:

ssh username@remote_host
2. Next, open the SSH daemon configuration file using a text editor of your choice:

sudo nano /etc/ssh/sshd_config
3. Look for the following line in the file:

PasswordAuthentication yes
4. Edit the configuration by changing the yes value to no. Thus, the directive should be as following:

PasswordAuthentication no
5. Save the file and exit the text editor.
6. To enable the changes, restart the sshd service using the command:

sudo systemctl restart sshd.service


## Reverse SSH


On the remote computer, we use the following command.

* The -R (reverse) option tells ssh that new SSH sessions must be created on the remote computer.
* The “43022:localhost:22” tells ssh that connection requests to port 43022 on the local computer should be forwarded to port 22 on the remote computer. Port 43022 was chosen because it is listed as being unallocated. It isn’t a special number.
* maksym@maks-it.com -p 2002 is the user account the remote computer is going to connect to on the local computer.

```bash
ssh -R 43022:localhost:22 maksym@maks-it.com -p 2002
```

You need to enable GatewayPorts=yes in the config for SSHd (/etc/ssh/sshd_config), not the client in order to enable binding to interfaces other than loopback on remote ports.

```bash
-o GatewayPorts=yes
```

Only works for local ports when passed to the ssh command.