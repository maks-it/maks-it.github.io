## What is DHCP?

DHCP stands for Dynamic Host Configuration Protocol. DHCP is a standardized network
protocol used on Internet Protocol networks for dynamically distributing network
configuration parameters, such as IP addresses for interfaces and services. DHCP Server
can be any server (Linux or Windows) that is used to distribute IP addresses automatically
to the clients in the network. Since, DHCP Server assigns IP addresses automatically to all
systems, a system or Network administrator need not to assign IP addresses manually to
every single machine in the network. DHCP is opt for system or Network administrator who
is managing thousands of systems.

In this tutorial, let us see how to install and configure DHCP Server in CentOS.

> A note of warning: Do not use two or more DHCP servers at the same time in your
> network. The client systems might not be able to get IP addresses from the multiple DHCP
> servers and it leads to IP address conflict issue. If your Router or Switch has DHCP feature
> enabled by default, you need to turn it off too.

More importantly, you must a assign a static IP address to your DHCP server’s network
interface card.

## Install DHCP Server in CentOS

First let us see how to install and configure DHCP server in CentOS 7 64bit.
Log in as root user.

1. install DHCP server on CentOS system, run:

```bash
dnf install dhcp-server
```

2. copy the sample dhcp configuration file to `/etc/dhcp/` directory.

```bash
cp /usr/share/doc/dhcp-server/dhcpd.conf.example /etc/dhcp/dhcpd.conf
```

3. edit `dhcpd.conf` file

```bash
nano /etc/dhcp/dhcpd.conf
```

```bash
# dhcpd.conf
#
# Configuration file for ISC dhcpd
#

# configuration of DNS update
update-static-leases on;

# https://lists.isc.org/pipermail/dhcp-users/2012-March/015039.html
# it sends an option to the client to tell it that the server
# will do the DNS updates, rather than the client.
# prevents
deny client-updates;

include "/etc/rndc.key";

# option definitions common to all supported networks...
option domain-name "corp.maks-it.com";
option domain-name-servers netserver.corp.maks-it.com;

default-lease-time 600;
max-lease-time 7200;

# Use this to enble / disable dynamic dns updates globally.
ddns-update-style interim;
ddns-domainname "corp.maks-it.com.";
ddns-rev-domainname "in-addr.arpa.";

# Filename that stores list of active IP lease allocations
lease-file-name "/var/lib/dhcpd/dhcpd.leases";

# If this DHCP server is the official DHCP server for the local
# network, the authoritative directive should be uncommented.
authoritative;

zone corp.maks-it.com. {
  primary 127.0.0.1;
  key rndc-key;
}

zone 0.0.168.192.in-addr.arpa. {
  primary 127.0.0.1;
  key rndc-key;
}

# Use this to send dhcp log messages to a different log file (you also
# have to hack syslog.conf to complete the redirection).
log-facility local7;

# No service will be given on this subnet, but declaring it helps the 
# DHCP server to understand the network topology.
# subnet 10.152.187.0 netmask 255.255.255.0 {
# }

# This is a very basic subnet declaration.
# subnet 10.254.239.0 netmask 255.255.255.224 {
#   range 10.254.239.10 10.254.239.20;
#   option routers rtr-239-0-1.example.org, rtr-239-0-2.example.org;
# }

# This declaration allows BOOTP clients to get dynamic addresses,
# which we don't really recommend.
# subnet 10.254.239.32 netmask 255.255.255.224 {
#   range dynamic-bootp 10.254.239.40 10.254.239.60;
#   option broadcast-address 10.254.239.31;
#   option routers rtr-239-32-1.example.org;
# }

# A slightly different configuration for an internal subnet.
subnet 192.168.0.0 netmask 255.255.255.0 {
  range 192.168.0.100 192.168.0.254;
  option domain-name-servers netserver.corp.maks-it.com;
  option domain-name "corp.maks-it.com";
  option routers 192.168.0.1;
  option broadcast-address 192.168.0.255;
  default-lease-time 600;
  max-lease-time 7200;
}

# Hosts which require special configuration options can be listed in
# host statements.   If no address is specified, the address will be
# allocated dynamically (if possible), but the host-specific information
# will still come from the host declaration.
# host passacaglia {
#   hardware ethernet 0:0:c0:5d:bd:95;
#   filename "vmunix.passacaglia";
#   server-name "toccata.example.com";
# }

# Fixed IP addresses can also be specified for hosts.   These addresses
# should not also be listed as being available for dynamic assignment.
# Hosts for which fixed IP addresses have been specified can boot using
# BOOTP or DHCP.   Hosts for which no fixed address is specified can only
# be booted with DHCP, unless there is an address range on the subnet
# to which a BOOTP client is connected which has the dynamic-bootp flag
# set.
# host fantasia {
#   hardware ethernet 08:00:07:26:c0:a5;
#   fixed-address fantasia.example.com;
# }

# You can declare a class of clients and then do address allocation
# based on that.   The example below shows a case where all clients
# in a certain class get addresses on the 10.17.224/24 subnet, and all
# other clients get addresses on the 10.0.29/24 subnet.
# class "foo" {
#   match if substring (option vendor-class-identifier, 0, 4) = "SUNW";
# }
# 
# shared-network 224-29 {
#   subnet 10.17.224.0 netmask 255.255.255.0 {
#     option routers rtr-224.example.org;
#   }
#   subnet 10.0.29.0 netmask 255.255.255.0 {
#     option routers rtr-29.example.org;
#   }
#   pool {
#     allow members of "foo";
#     range 10.17.224.10 10.17.224.250;
#   }
#   pool {
#     deny members of "foo";
#     range 10.0.29.10 10.0.29.230;
#   }
# }
```

## Star DHCP server

After making all the changes you want, save and close the file. Be mindful that if you have
another unused entries on the `dhcpd.conf` file, comment them. Otherwise, you’ll have
issues while starting `dhcpd` service.
Now, start the `dhcpd` service and make it to start automatically on every reboot.
On CentOS 7.x systems:
```
systemctl enable dhcpd
systemctl start dhcpd
```

## DHCP Parameters explanation

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
