# What is DHCP?

DHCP stands for Dynamic Host Configuration Protocol. DHCP is a standardized network
protocol used on Internet Protocol networks for dynamically distributing network
configuration parameters, such as IP addresses for interfaces and services. DHCP Server
can be any server (Linux or Windows) that is used to distribute IP addresses automatically
to the clients in the network. Since, DHCP Server assigns IP addresses automatically to all
systems, a system or Network administrator need not to assign IP addresses manually to
every single machine in the network. DHCP is opt for system or Network administrator who
is managing thousands of systems.

In this tutorial, let us see how to install and configure DHCP Server in CentOS.

>>>
A note of warning: Do not use two or more DHCP servers at the same time in your
network. The client systems might not be able to get IP addresses from the multiple DHCP
servers and it leads to IP address conflict issue. If your Router or Switch has DHCP feature
enabled by default, you need to turn it off too.
>>>

More importantly, you must a assign a static IP address to your DHCP server’s network
interface card.

# Install DHCP Server in CentOS

First let us see how to install and configure DHCP server in CentOS 7 64bit.
Log in as root user.
1. install DHCP server on CentOS system, run:

``` bash
yum install dhcp
```

2. copy the sample dhcp configuration file to `/etc/dhcp/` directory.

``` bash
cp /usr/share/doc/dhcp-server/dhcpd.conf.example /etc/dhcp/dhcpd.conf
```

3. edit `dhcpd.conf` file

```
nano /etc/dhcp/dhcpd.conf
```

# Star DHCP server

After making all the changes you want, save and close the file. Be mindful that if you have
another unused entries on the `dhcpd.conf` file, comment them. Otherwise, you’ll have
issues while starting `dhcpd` service.
Now, start the `dhcpd` service and make it to start automatically on every reboot.
On CentOS 7.x systems:
```
systemctl enable dhcpd
systemctl start dhcpd
```

# DHCP Parameters explanation

Parameter|Definition
--- | ---
ddns-update-style|Type of DDNS update to use with local DNS Server
ignore client-updates|Ignore all client requests for DDNS update
lease-file-name|Filename that stores list of active IP lease allocations
authoritative|Set as master server, protects against rogue DHCP servers and misconfigured clients
option domain-name|Specifies the Internet Domain Name to append to a client's hostname
option domain-name-servers|The DNS servers the clients should use for name resolution
default-lease-time|The default time in seconds that the IP is leased
max-lease-time|The max time in seconds that the IP is leased
option routers|Specifies the Gateway for the client to use
option subnet-mask|The subnet mask specific to the lease range
option broadcast-address|The broadcast address specific to the lease range
option ntp-servers|Network Time Protocol servers available to the clients
option netbios-name-server|The NetBIOS name server (WINS)
option netbios-node-type|The NetBIOS name resolution method (8=hybrid)
range|The range of valid IP addresses available for client offer

