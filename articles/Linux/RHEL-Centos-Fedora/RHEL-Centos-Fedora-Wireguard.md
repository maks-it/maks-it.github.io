# Set up WireGuard VPN server

WireGuard is a free, open-source modern and fast VPN with state-of-the-art cryptography. It is quicker and simpler as compared to IPSec and OpenVPN. Originally, released for the Linux kernel, but it is getting cross-platform support for other operating systems too. This page explains how to install and set up WireGuard VPN on CentOS 8 Linux.


## Step 1 – Update your system

```bash
sudo dnf update
```

## Step 2 – Enable and install EPEL repo

We need to install a package named wireguard-tools from EPEL repo that provides the wg binary for controlling the WireGuard server.

```bash
sudo dnf install epel-release
```

Make sure you enable the PowerTools repository since EPEL packages may depend on packages from it:

```bash
sudo dnf install 'dnf-command(config-manager)'
sudo dnf config-manager --set-enabled PowerTools
```

## Step 3 – Set up wireguard repo

Before we can install the wireguard Linux kernel module, turn on the official wireguard repo. Execute the following command:

```bash
sudo dnf copr enable jdoss/wireguard
```

## Step 4 – Installing a WireGuard VPN server on CentOS 8

Now we got everything set up. It is time for setting up a WireGuard VPN server on CentOS 8 box. Run:

```bash
sudo dnf install wireguard-dkms wireguard-tools
```

The above will also install the GNU GCC compiler collection to compile and build the required Linux kernel modules. So it would be best if you waited some time to complete the procedure.

```bash
sudo mkdir -v /etc/wireguard/
sudo sh -c 'umask 077; touch /etc/wireguard/wg0.conf'
sudo ls -l /etc/wireguard/wg0.conf
```

### Create a private and public key pair for the WireGuard server

We need to run the following commands in /etc/wireguard/ directory. Use the cd command:

```bash
cd /etc/wireguard/
```

Run the following command:

```bash
sudo sh -c 'umask 077; wg genkey | tee privatekey | wg pubkey > publickey'
```

To view keys use the cat command and ls command:

```bash
ls -l privatekey publickey
sudo cat privatekey
#### Note down the privatekey ####
sudo cat publickey
```

### Configure WireGuard server

Edit the /etc/wireguard/wg0.conf file:

```bash
sudo nano /etc/wireguard/wg0.conf
```

Append the following directives:

```bash
[Interface]
## VPN server private IP address ##
Address = 192.168.5.1/24
 
## VPN server port ##
ListenPort = 31194
 
## VPN server's private key i.e. /etc/wireguard/privatekey ##
PrivateKey = cBRd8MhXckz5ZnNRvk8F+r0558sfJlI0YdJ7ZQfWqUM=
 
## Save and update this config file when a new peer (vpn client) added ##
SaveConfig = true
```

Save and close the file.

### Set up firewalld rules

We have set up a firewall using FirewallD on our CentOS 8 server. Hence, we need to open UDP port 31194 by creating wireguard service:

```bash
sudo nano /etc/firewalld/services/wireguard.xml
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>wireguard</short>
  <description>WireGuard open UDP port 31194 for client connections</description>
  <port protocol="udp" port="31194"/>
</service>
```

Next, I am going to enable our WireGuard service in firewalld using the firewall-cmd command as follows:

```bash
sudo firewall-cmd --permanent --add-service=wireguard --zone=public
```

Turn on masquerading so all traffic coming and going out from 192.168.5.0/24 routed correctly via our public IP address 172.105.120.136/24:

```bash
sudo firewall-cmd --permanent --zone=public --add-masquerade
```

Finally reload the firewalld and list our configuration:

```bash
sudo firewall-cmd --reload
sudo firewall-cmd --list-all
```

Sample outputs:

```bash
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: eth0
  sources: 
  services: wireguard ssh
  ports: 
  protocols: 
  masquerade: yes
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules:
```

### Turn on IPv4/IPv6 forwarding

Edit the /etc/sysctl.d/99-sysctl.conf file

```bash
sudo vi /etc/sysctl.d/99-sysctl.conf
```

Append the following config options:

```bash
## Turn on bbr ##
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr
 
## for IPv4 ##
net.ipv4.ip_forward = 1
 
## Turn on basic protection/security ##
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.all.rp_filter = 1
net.ipv4.tcp_syncookies = 1
 
## for IPv6 - uncomment the following line ##
#net.ipv6.conf.all.forwarding = 1
```

Reload changes using the sysctl command:

```bash
 sudo sysctl -p
```

### Allow peer to peer connection

By default, firewalld will drop all communication between internal (wg0) and the public network (eth0). So we are going to add interface wg0 to the internal network and turn on masquerading as follows

```bash
sudo firewall-cmd --add-interface=wg0 --zone=internal
sudo firewall-cmd --permanent --zone=internal --add-masquerade
```

## Step 5 – Enable and start WireGuard service

Now we installed and configured server correctly it is time to enable and start wireguard service using the systemctl command:

```bash
sudo systemctl enable wg-quick@wg0 #<-- turn it on
sudo systemctl start wg-quick@wg0 #<-- start it
sudo systemctl status wg-quick@wg0 #<-- get status
```

Verify that interface wg0 is up and running using the ip command:

```bash
sudo wg
sudo ip a show wg0
```

Please note down the public key when displayed.

## Step 6 – Wireguard VPN client configuration (Fedora)

```bash
sudo dnf install wireguard-tools
```

Let us create our VPN client config:

```bash
sudo mkdir -v /etc/wireguard/
sudo sh -c 'umask 077; touch /etc/wireguard/wg0.conf'
sudo ls -l /etc/wireguard/wg0.conf
cd /etc/wireguard/
sudo sh -c 'umask 077; wg genkey | tee privatekey | wg pubkey > publickey'
#### Note down the privatekey ####
sudo cat privatekey
```

Edit the /etc/wireguard/wg0.conf file:

```bash
sudo nano /etc/wireguard/wg0.conf
```

Append the following directives:

```bash
[Interface]
## client private key ##
PrivateKey = sKVVArGeo75fCkskltZc8WRIi9fPQ3SPLXmrr8uBp3M=
 
## client ip address ##
Address = 192.168.5.2/24
 
[Peer]
## CentOS 8 server public key ##
PublicKey = qdjdqh2+N3DEMDUDRob8K3b+9BZFJbT59f+rBrl99zM
 
## set ACL ##
#AllowedIPs = 192.168.5.0/24
## Turn on NAT for client so internet routed thorugh our vpn
AllowedIPs = 0.0.0.0/0
 
## Your CentOS 8 server's public IPv4/IPv6 address and port ##
Endpoint = 172.105.120.136:31194
 
##  Key connection alive ##
PersistentKeepalive = 15
```

Enable and start VPN client/peer, run:

```bash
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0
sudo systemctl status wg-quick@wg0
```

### Allow client and server connection

We need to configure the server-side and allow a connection between the client and the server. Let us go back to our CentOS 8 server and edit wg0.conf file to add [Peer] (client) information as follows (type commands on the server):

```bash
{vivek@centos8:~ }$ sudo systemctl stop wg-quick@wg0
{vivek@centos8:~ }$ sudo nano /etc/wireguard/wg0.conf
```

Append the following config:

```bash
[Peer]
## client VPN public key ##
PublicKey = hM/J1IVUns1F4gWjA11pOPq6uDmlYsSq0o7JWCQ02C4=
 
## client VPN IP address (note /32 subnet) ##
AllowedIPs = 192.168.5.2/32
```

Save and close the file in nano. Next start the service again, run:

```bash
sudo systemctl start wg-quick@wg0
```

## Verification

That is all. By now, both servers and clients must be connected securely using VPN. Let us test the connection. Type the following ping command on your client machine:

```bash
{vivek@centos8-vpn-client:~ }$ ping -c 4 192.168.5.1
{vivek@centos8-vpn-client:~ }$ sudo wg
```