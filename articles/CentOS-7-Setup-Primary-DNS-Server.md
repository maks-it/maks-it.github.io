# What is DNS?

The Domain Name Systems (DNS) is the phonebook of the Internet. Humans access information online through domain names, like nytimes.com or espn.com. Web browsers interact through Internet Protocol (IP) addresses. DNS translates domain names to IP addresses so browsers can load Internet resources.

Each device connected to the Internet has a unique IP address which other machines use to find the device. DNS servers eliminate the need for humans to memorize IP addresses such as 192.168.1.1 (in IPv4), or more complex newer alphanumeric IP addresses such as 2400:cb00:2048:1::c629:d7a2 (in IPv6).

In this tutorial, let us see how to install and configure DHCP Server in CentOS.

# DNS Servers to configure into the network:
```
Operating System     : CentOS 7
Hostname             : masterdns.nc.local
IP Address           : 192.168.0.1/24
```

```
Operating System     : Router
Hostname             : slavedns.nc.local
IP Address           : 192.168.0.2/24
```

# Install DHCP Server in CentOS

1. install DNC server on CentOS system, run:
```
yum install bind bind-utils -y
```
2. configure DNS
Edit named.conf:
```
nano /etc/named.conf
```
3. create Zone files (are necessary to setup forward and reverse zones).

3.1. Forward Zone file:
```
nano /var/named/forward.nc
```

3.2 Reverse Zone file:
```
nano /var/named/reverse.nc
```

4. add firewall rules:
```
firewall-cmd --permanent --add-port=53/tcp
firewall-cmd --permanent --add-port=53/udp
```
5. restart firewall:
```
firewall-cmd --reload
```

6. configuring Permissions, Ownership, and SELinux

```
chgrp named -R /var/named
chown -v root:named /etc/named.conf
restorecon -rv /var/named
restorecon /etc/named.conf
```

# Test DNS Server Configuration Files

Test DNS configuration and zone files for any syntax errors

1. check DNS default configuration file:
```
named-checkconf /etc/named.conf
```
If it returns nothing, your configuration file is valid.


2. check forward zone:
```
named-checkzone nc.local /var/named/forward.nc
```

sample output:
```
zone nc.local/IN: loaded serial 2011071001
OK
```

3. check reverse zone:
```
named-checkzone nc.local /var/named/reverse.nc
```

sample output:
```
zone nc.local/IN: loaded serial 2011071001
OK
```

# Start the DNS service

```
systemctl enable named
systemctl start named
```

# Add the DNS Server details in your network interface config file

1. edit interface configuration file:
```
nano /etc/sysconfig/network-scripts/ifcfg-ens33
```

>>>
ifcfg-ens33 - your interface name may have different name
>>>
Add:
```
DNS1="192.168.0.1"
```

2. edit `resolv.conf`:
```
nano /etc/resolv.conf
```

Add the name server ip address:
```
nameserver      192.168.0.1
```

Restart network service:
```
systemctl restart network
```


# Test DNS Server

```
dig masterdns.nc.local
```

```
; <<>> DiG 9.9.4-RedHat-9.9.4-14.el7 <<>> masterdns.nc.local
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 25179
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;masterdns.nc.local.    IN    A

;; ANSWER SECTION:
masterdns.nc.local. 86400    IN    A    192.168.0.1

;; AUTHORITY SECTION:
nc.local.        86400    IN    NS    secondarydns.nc.local.
nc.local.        86400    IN    NS    masterdns.nc.local.

;; ADDITIONAL SECTION:
secondarydns.nc.local. 86400 IN    A    192.168.0.2

;; Query time: 0 msec
;; SERVER: 192.168.0.1#53(192.168.0.1)
;; WHEN: Wed Aug 20 16:20:46 IST 2018
;; MSG SIZE  rcvd: 125
```



```
nslookup nc.local
```


```
erver:        192.168.0.1
Address:    192.168.0.1#53

Name:    nc.local
Address: 192.168.0.1
Name:    nc.local
Address: 192.168.0.2
```



Now the Primary DNS server is ready to use.


