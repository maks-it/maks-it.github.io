## What is DNS?

The Domain Name Systems (DNS) is the phonebook of the Internet. Humans access information online through domain names, like nytimes.com or espn.com. Web browsers interact through Internet Protocol (IP) addresses. DNS translates domain names to IP addresses so browsers can load Internet resources.

Each device connected to the Internet has a unique IP address which other machines use to find the device. DNS servers eliminate the need for humans to memorize IP addresses such as 192.168.1.1 (in IPv4), or more complex newer alphanumeric IP addresses such as 2400:cb00:2048:1::c629:d7a2 (in IPv6).

In this tutorial, let us see how to install and configure DHCP Server in CentOS.

## DNS Servers to configure into the network:

```
Operating System     : Fedora Server
Hostname             : netserver.corp.maks-it.com
IP Address           : 192.168.0.2/24
```

```
Operating System     : Router
Hostname             : router.corp.maks-it.com
IP Address           : 192.168.0.1/24
```

## Install DNS Server in RHEL/CentOS/Fedora

1. install DNC server on CentOS system, run:

```bash
dnf install bind bind-utils -y
```


2. Generate RNDC key (but probably it already exists)

```bash
sudo rndc-confgen
```

```bash
# Start of rndc.conf
key "rndc-key" {
        algorithm hmac-sha256;
        secret "FswCGbOmsP0rtjk3UWteD0J4TxJZrVr3YJxt5u4lKA0=";
};

options {
        default-key "rndc-key";
        default-server 127.0.0.1;
        default-port 953;
};
# End of rndc.conf

# Use with the following in named.conf, adjusting the allow list as needed:
# key "rndc-key" {
#       algorithm hmac-sha256;
#       secret "FswCGbOmsP0rtjk3UWteD0J4TxJZrVr3YJxt5u4lKA0=";
# };
# 
# controls {
#       inet 127.0.0.1 port 953
#               allow { 127.0.0.1; } keys { "rndc-key"; };
# };
# End of named.conf
```

Insert the generated RNDC configuration stanza into the file `/etc/rndc.key`. Your code will be different:

```bash
key "rndc-key" {
        algorithm hmac-sha256;
        secret "FswCGbOmsP0rtjk3UWteD0J4TxJZrVr3YJxt5u4lKA0=";
};
```


3. configure DNS

Edit `named.conf`:

```bash
nano /etc/named.conf
```

```bash
//
// named.conf
//
// Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
// server as a caching only nameserver (as a localhost DNS resolver only).
//
// See /usr/share/doc/bind*/sample/ for example named configuration files.
//

options {
        listen-on port 53 { 127.0.0.1; 192.168.0.2; };
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        recursing-file  "/var/named/data/named.recursing";
        secroots-file   "/var/named/data/named.secroots";
        allow-query     { localhost; 192.168.0.0/24; };
        allow-transfer  { localhost; 192.168.0.1; };

        /*
         - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion.
         - If you are building a RECURSIVE (caching) DNS server, you need to enable
           recursion.
         - If your recursive DNS server has a public IP address, you MUST enable access
           control to limit queries to your legitimate users. Failing to do so will
           cause your server to become part of large scale DNS amplification
           attacks. Implementing BCP38 within your network would greatly
           reduce such attack surface
        */
        recursion yes;

        dnssec-validation yes;

        managed-keys-directory "/var/named/dynamic";
        geoip-directory "/usr/share/GeoIP";

        pid-file "/run/named/named.pid";
        session-keyfile "/run/named/session.key";

        /* https://fedoraproject.org/wiki/Changes/CryptoPolicy */
        include "/etc/crypto-policies/back-ends/bind.config";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "." IN {
        type hint;
        file "named.ca";
};

include "/etc/rndc.key"
controls {
       inet 127.0.0.1 port 953
       allow { 127.0.0.1; } keys { "rndc-key"; };
};

zone "corp.maks-it.com" IN {
        type master;
        file "/var/lib/named/forward.nc";
        allow-update { key rndc-key; };
};

zone "0.168.192.in-addr.arpa" IN {
        type master;
        file "/var/lib/named/reverse.nc";
        allow-update { key rndc-key; };
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";

```

4. Forward Zone file:

```bash
nano /var/lib/named/forward.nc
```

```bash
$TTL 86400
@       IN  SOA     netserver.corp.maks-it.com. root.corp.maks-it.com. (
        1       ;Serial
        3600    ;Refresh
        1800    ;Retry
        604800  ;Expire
        86400   ;Minimum TTL
)

;Name Servers
@       IN  NS          netserver.corp.maks-it.com.
@       IN  A           192.168.0.2

netserver       IN  A   192.168.0.2
_vlmcs._tcp.corp.maks-it.com. 3600 IN SRV 10 0 1688 netserver.corp.maks-it.com.

;Other
srvweb0001      IN  A   192.168.0.3
srvweb0002      IN  A   192.168.0.6
srvsql0001      IN  A   192.168.0.4


;Mail Server
srvmta0001      IN  A   192.168.0.5
@        IN MX  10      srvmta0001.corp.maks-it.com.
```

```bash
$ORIGIN .
$TTL 86400      ; 1 day
corp.maks-it.com        IN SOA  netserver.corp.maks-it.com. root.corp.maks-it.com. (
                                1       ; serial
                                3600    ; refresh (1 hour)
                                1800    ; retry (30 minutes)
                                604800  ; expire (1 week)
                                86400   ; minimum (1 day)
                                )
                        NS      netserver.corp.maks-it.com.
$ORIGIN corp.maks-it.com.
$TTL 86400      ; 1 day
netserver               A       192.168.0.2
```

5. Reverse Zone file:

```bash
nano /var/lib/named/reverse.nc
```

```bash
$TTL 86400
@       IN  SOA     netserver.corp.maks-it.com. root.corp.maks-it.com. (
        1       ;Serial
        3600    ;Refresh
        1800    ;Retry
        604800  ;Expire
        86400   ;Minimum TTL
)

@       IN  NS          netserver.corp.maks-it.com.
@       IN  PTR         corp.maks-it.com.

netserver       IN  A   192.168.0.2

101     IN  PTR         netserver.corp.maks-it.com.
```

6. Zone files write permissions



```bash
sudo chown named:named /var/lib/named -R 
```

Fix Selinux `named` rules to write on zone files:

```bash
semanage fcontext --add --type named_zone_t '/var/lib/named(/.*)?'
restorecon -rvF /var/lib/named
```

check if everything is fine:

```bash
semanage fcontext -l | grep '/var/lib/named'
```

your output should be like this:

```bash
/var/lib/named(/.*)?    all files    system_u:object_r:named_zone_t:s0 
```

and

```bash
ls -lZ /var/lib/named

total 12
-rw-r--r--. 1 named named system_u:object_r:named_zone_t:s0  692 Mar 31 21:55 forward.nc
-rw-r--r--. 1 named named system_u:object_r:named_zone_t:s0 3078 Mar 31 21:43 forward.nc.jnl
-rw-rw-r--. 1 named named system_u:object_r:named_zone_t:s0  545 Mar 29 21:08 reverse.nc
```

1. add firewall rules:

```bash
firewall-cmd --permanent --add-port=53/tcp
firewall-cmd --permanent --add-port=53/udp
```

8. restart firewall:
   
```bash
firewall-cmd --reload
```

9. configuring Permissions, Ownership, and SELinux

```bash
chgrp named -R /var/named
chown -v root:named /etc/named.conf
restorecon -rv /var/named
restorecon /etc/named.conf
```

## Test DNS Server Configuration Files

Test DNS configuration and zone files for any syntax errors

1. check DNS default configuration file:
   
```bash
named-checkconf /etc/named.conf
```

If it returns nothing, your configuration file is valid.


2. check forward zone:

```bash
named-checkzone corp.maks-it.com /var/named/forward.nc
```

sample output:

```bash
zone corp.maks-it.com/IN: loaded serial 2011071001
OK
```

3. check reverse zone:

```bash
named-checkzone corp.maks-it.com /var/named/reverse.nc
```

sample output:

```bash
zone corp.maks-it.com/IN: loaded serial 2011071001
OK
```

## Start the DNS service

```bash
systemctl enable named
systemctl start named
```

## Add the DNS Server details in your network interface config file

1. edit interface configuration file:

```bash
nano /etc/sysconfig/network-scripts/ifcfg-ens33
```

>>>
ifcfg-ens33 - your interface name may have different name
>>>
Add:

```bash
DNS1="192.168.0.2"
```

2. edit `resolv.conf`:
   
```bash
nano /etc/resolv.conf
```

Add the name server ip address:

```bash
nameserver      192.168.0.1
```

Restart network service:

```bash
systemctl restart network
```


## Test DNS Server

```bash
dig netserver.corp.maks-it.com
```

```bash
; <<>> DiG 9.16.27-RH <<>> netserver.corp.maks-it.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 51058
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 4b5d865a6810c82b0100000062435b93faec9e00a4473b07 (good)
;; QUESTION SECTION:
;netserver.corp.maks-it.com.    IN      A

;; ANSWER SECTION:
netserver.corp.maks-it.com. 86400 IN    A       192.168.0.2

;; Query time: 0 msec
;; SERVER: 192.168.0.2#53(192.168.0.2)
;; WHEN: Tue Mar 29 21:18:43 CEST 2022
;; MSG SIZE  rcvd: 99
```

```bash
nslookup corp.maks-it.com
```

```bash
Server:         192.168.0.2
Address:        192.168.0.2#53

Name:   corp.maks-it.com
Address: 192.168.0.2
Name:   corp.maks-it.com
Address: 192.168.0.1
```

Now the Primary DNS server is ready to use.
