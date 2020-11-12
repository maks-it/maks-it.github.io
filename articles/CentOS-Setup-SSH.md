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