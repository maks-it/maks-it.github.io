# How To Set Up WireGuard Firewall Rules in Linux

How do I set up WireGuard Firewall rules (iptables) in Linux?

For road warrior WireGuard and other purposes, you need to set up and configure firewall rules. You need to configure NAT (Network Address Translation) to allow WireGuard clients to access the Internet. In Linux, we use a term called IP Masquerade. It means one to many NAT (1:Many). We also need a FORWARD chain rule. This page explains how to set up NAT and FORWARD firewall rules for WireGuard in Linux.

## Procedure to set up WireGuard firewall rules

Linux comes with raw iptables and easy to use frontend scripts. For example, UFW is one such popular tool. Before we use any tools, we need to understand the exact iptables rules. Naturally, you must have WireGuard configured.

## Step 1: Setting up NAT firewall rules

The syntax is as follows:

```bash
iptables -t nat -I POSTROUTING 1 -s {sub/net} -o {interface} -j MASQUERADE
```

Make sure all outgoing packets are translated via VPN:

```bash
iptables -t nat -I POSTROUTING 1 -s 10.8.1.0/24 -o eth0 -j MASQUERADE
```

Where,

- -t nat : Set up nat table for WireGuard.
- -I POSTROUTING 1 : Insert rule at position 1 for altering packets as they are about to go out for the POSTROUTING chain.
- -s 10.8.1.0/24 : Only do NAT if source address created by WireGuard wg0 interface.
- -o eth0 : Name of an interface via which a packet is going to be sent. In this case, eth0 connected to the Internet.
- -j MASQUERADE : Tell (jump) what to do if the packet matches according to given conditions. The MASQUERADE target is only valid in the nat table, in the POSTROUTING chain. This rule is responsible for routing traffic to the Internet for all WireGuard clients.

## Step 2: Accept all traffic created by wg0 interface

Allow all traffic on wg0 interface:

```bash
iptables -I INPUT 1 -i {interface} -j ACCEPT
iptables -I INPUT 1 -i wg0 -j ACCEPT
```

The above rules allows for packets destined to wg0.

## Step 3: Configuring FORWARD rules

We must allow for packets being routed through the WireGuard server by setting up the FORWARD rule. The syntax is:

```bash
iptables -I FORWARD 1 -i eth0 -o wg0 -j ACCEPT
iptables -I FORWARD 1 -i wg0 -o eth0 -j ACCEPT
```

## Step 4: Open WireGuard UDP port # 51194

Finally, open UDP port # 51194 as follows:

```bash
iptables -I INPUT 1 -i eth0 -p udp --dport 51194 -j ACCEPT
```

## Step 5: Command to remove WireGuard iptables rules

We can reverse all command by deleting all added iptabes rules as follows:

```bash
iptables -t nat -D POSTROUTING -s 10.8.1.0/24 -o eth0 -j MASQUERADE
iptables -D INPUT -i wg0 -j ACCEPT
iptables -D FORWARD -i eth0 -o wg0 -j ACCEPT
iptables -D FORWARD -i wg0 -o eth0 -j ACCEPT
iptables -D INPUT -i eth0 -p udp --dport 51194 -j ACCEPT
```

## Step 6: Turn on IP forwarding on Linux

For IPv4 we set the following Linux kernel variables to accept incoming network packets on wg0, passed on to another network interface such as eth0, and then forwards it accordingly:

```bash
sysctl -w net.ipv4.ip_forward=1
```

## For IPv6, try the following sysctl command:

```bash
sysctl -w net.ipv6.conf.all.forwarding=1
```

## Step 7: Update wireguard config files for firewall and routing support

We need to tell WireGuard commands and script snippets which will be executed by using the following two directives:

```bash
# Turn on NAT when wg0 comes up #
PostUp = /path/to/add-nat-routing.sh
# Turn of NAT when wg0 goes down #
PostDown = /path/to/remove-nat-routing.sh
```

## Putting it all together: Firewall rule for WireGuard

Update your /etc/wireguard/wg0.conf file as follows:

```bash
nano /etc/wireguard/wg0.conf
```

Append in the [Interface] section:

```bash
PostUp = /etc/wireguard/helper/add-nat-routing.sh
PostDown = /etc/wireguard/helper/remove-nat-routing.sh
```

Here is how it should look:

```bash
[Interface]
Address = 10.8.0.1/24
ListenPort = 51194
PrivateKey = My_Server_KEY
SaveConfig = false
PostUp = /etc/wireguard/helper/add-nat-routing.sh
PostDown = /etc/wireguard/helper/remove-nat-routing.sh
 
[Peer]
# My Linux Desktop 
PublicKey = My_Linux_Desktop_KEY
PresharedKey = My_PRE_SHARED_KEY
AllowedIPs = 10.8.0.2/32
```

Create a new directory using the mkdir command:

```bash
mkdir -v /etc/wireguard/helper/
```

Contains of add-nat-routing.sh displayed using the cat command:

```bash
cat /etc/wireguard/helper/add-nat-routing.sh
```

```bash
#!/bin/bash
IPT="/sbin/iptables"
IPT6="/sbin/ip6tables"          
 
IN_FACE="eth0"                   # NIC connected to the internet
WG_FACE="wg0"                    # WG NIC 
SUB_NET="10.8.1.0/24"            # WG IPv4 sub/net aka CIDR
WG_PORT="51194"                  # WG udp port
SUB_NET_6="fd42:42:42:42::/112"  # WG IPv6 sub/net
 
## IPv4 ##
$IPT -t nat -I POSTROUTING 1 -s $SUB_NET -o $IN_FACE -j MASQUERADE
$IPT -I INPUT 1 -i $WG_FACE -j ACCEPT
$IPT -I FORWARD 1 -i $IN_FACE -o $WG_FACE -j ACCEPT
$IPT -I FORWARD 1 -i $WG_FACE -o $IN_FACE -j ACCEPT
$IPT -I INPUT 1 -i $IN_FACE -p udp --dport $WG_PORT -j ACCEPT
 
## IPv6 (Uncomment) ##
## $IPT6 -t nat -I POSTROUTING 1 -s $SUB_NET_6 -o $IN_FACE -j MASQUERADE
## $IPT6 -I INPUT 1 -i $WG_FACE -j ACCEPT
## $IPT6 -I FORWARD 1 -i $IN_FACE -o $WG_FACE -j ACCEPT
## $IPT6 -I FORWARD 1 -i $WG_FACE -o $IN_FACE -j ACCEPT
```

AND:

```bash
cat /etc/wireguard/helper/remove-nat-routing.sh
```

```bash
#!/bin/bash
IPT="/sbin/iptables"
IPT6="/sbin/ip6tables"          
 
IN_FACE="eth0"                   # NIC connected to the internet
WG_FACE="wg0"                    # WG NIC 
SUB_NET="10.8.1.0/24"            # WG IPv4 sub/net aka CIDR
WG_PORT="51194"                  # WG udp port
SUB_NET_6="fd42:42:42:42::/112"  # WG IPv6 sub/net
 
# IPv4 rules #
$IPT -t nat -D POSTROUTING -s $SUB_NET -o $IN_FACE -j MASQUERADE
$IPT -D INPUT -i $WG_FACE -j ACCEPT
$IPT -D FORWARD -i $IN_FACE -o $WG_FACE -j ACCEPT
$IPT -D FORWARD -i $WG_FACE -o $IN_FACE -j ACCEPT
$IPT -D INPUT -i $IN_FACE -p udp --dport $WG_PORT -j ACCEPT
 
# IPv6 rules (uncomment) #
## $IPT6 -t nat -D POSTROUTING -s $SUB_NET_6 -o $IN_FACE -j MASQUERADE
## $IPT6 -D INPUT -i $WG_FACE -j ACCEPT
## $IPT6 -D FORWARD -i $IN_FACE -o $WG_FACE -j ACCEPT
## $IPT6 -D FORWARD -i $WG_FACE -o $IN_FACE -j ACCEPT
```

Make sure you create the following file using a text editor too:

```bash
nano /etc/sysctl.d/10-wireguard.conf
```

Add the following text:

```bash
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1
```

Reload all changes and turn on NAT routing:


```bash
sysctl -p /etc/sysctl.d/10-wireguard.conf
chmod -v +x /etc/wireguard/helper/*.sh
systemctl restart wg-quick@wg0.service
```

### Update client’s config file

You must tell Wireguard client that the remote server is the client’s gateway. In other words we are going to override the default route on the client. Edit the /etc/wireguard/wg0.conf on client side as follows in [Peer] section.

```bash
AllowedIPs = 0.0.0.0/0
```

Here is how it looks on client side:

```bash
# Config client for Linode VPN #
[Interface]
## This Desktop/client's private key ##
PrivateKey = {KEY_GOES_HERE}
 
## Client ip address ##
Address = 10.8.0.2/24
 
[Peer]
## Remote Ubuntu 20.04 wg0 server public key ##
PublicKey = {KEY_GOES_HERE}
 
## set ACL ##
#################################################
## Allow remote server as gateway 
## Edit/Update old AllowedIPs entry as follows 
## Otherwise client won't show server's IP 
#################################################
AllowedIPs = 0.0.0.0/0
 
## Your Ubuntu 20.04 LTS server's public IPv4/IPv6 address and port ##
Endpoint = {SERVER_IP_HERE}:51194
 
##  Key connection alive ##
PersistentKeepalive = 15
```

### Verification

Test your configuration from the client side. See if you can access the Internet using the ping command, dig command/host command and a web-browser:

```bash
vivek@client:~$ ping -c 4 1.1.1.1
vivek@client:~$ host www.cyberciti.biz
## See if you can access WG based DNS server too (must be configured) ##
vivek@client:~$ dig -p 53 www.google.com 10.8.0.1
## View routing using the ip command ##
vivek@client:~$ ip r
## We must get our WireGuard public IP address ##
## Find public IP address from command line on Linux ##
vivek@client:~$ dig TXT +short o-o.myaddr.l.google.com @ns1.google.com
```

## Conclusion

In this guide, we have shown you how to enable IP forwarding and NAT rules using iptables in Linux for WireGuard VPN clients to provide internal clients with Internet access. See WireGuard home page for more information.
