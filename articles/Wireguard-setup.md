First we need to create a private and public key pair for the WireGuard server. Let us cd into /etc/wireguard/ directory using the cd command as follows:
## Wireguard VPN server configuration

```bash
$ sudo -i
# cd /etc/wireguard/
```

Execute the following command:
```bash
# umask 077; wg genkey | tee privatekey | wg pubkey > publickey
```
To view keys created use the cat command and ls command:
```bash
# ls -l privatekey publickey
# cat privatekey
## Please note down the private key ##
# cat publickey
```

Set Up WireGuard VPN on Debian by Editing wg0.conf
Edit or update the /etc/wireguard/wg0.conf file as follows:
```bash
$ sudo nano /etc/wireguard/wg0.conf
```

Append the following config directives:
```bash
## Set Up WireGuard VPN on Debian By Editing/Creating wg0.conf File ##
[Interface]
## My VPN server private IP address ##
Address = 192.168.10.1/24
 
## My VPN server port ##
ListenPort = 51194
 
## VPN server's private key i.e. /etc/wireguard/privatekey ##
PrivateKey = eEvqkSJVw/7cGUEcJXmeHiNFDLBGOz8GpScshecvNHU
 
## Save and update this config file when a new peer (vpn client) added ##
SaveConfig = true
```

Turn the WireGuard service at boot time using the systemctl command, run:
```bash
$ sudo systemctl enable wg-quick@wg0
```

Start the service, execute:
```bash
$ sudo systemctl start wg-quick@wg0
```

Get the service status, run:
```bash
$ sudo systemctl status wg-quick@wg0
```

Verify that interface named wg0 is up and running on Debian server using the ip command:
```bash
$ sudo wg
$ sudo ip a show wg0
```

## Wireguard VPN client configuration

Next we need create VPN client config on Debian/Debian/CentOS Linux destkop:

```bash
sudo sh -c 'umask 077; touch /etc/wireguard/wg0.conf'
$ sudo -i
# cd /etc/wireguard/
# umask 077; wg genkey | tee privatekey | wg pubkey > publickey
# ls -l publickey privatekey
## Note down the privatekey ##
# cat privatekey
```

Edit the /etc/wireguard/wg0.conf file:
```bash
$ sudo nano /etc/wireguard/wg0.conf
```

Append the following directives:
```bash
[Interface]
## This Desktop/client's private key ##
PrivateKey = uJPzgCQ6WNlAUp3s5rabE/EVt1qYh3Ym01sx6oJI0V4
 
## Client ip address ##
Address = 192.168.10.2/24
 
[Peer]
## Debian 10 server public key ##
PublicKey = qdjdqh2pN3DEMDUDRob8K3bp9BZFJbT59fprBrl99zM
 
## set ACL ##
AllowedIPs = 192.168.10.0/24
 
## Your Debian 10 LTS server's public IPv4/IPv6 address and port ##
Endpoint = 172.105.112.120:51194
 
##  Key connection alive ##
PersistentKeepalive = 20
```

Enable and start VPN client/peer connection, run:
```bash
$ sudo systemctl enable wg-quick@wg0
$ sudo systemctl start wg-quick@wg0
$ sudo systemctl status wg-quick@wg0
```

Allow desktop client and Debian server connection over VPN (peer)
We need to configure the server-side peer-to-peer VPN option and allow a connection between the Desktop client computer and the server. Let us go back to our Debian 10 LTS server and edit the wg0.conf file to add [Peer] (client) information as follows (type commands on your server box):
```bash
$ sudo systemctl stop wg-quick@wg0
$ sudo vi /etc/wireguard/wg0.conf
```

Append the following config:
```bash
[Peer]
## Desktop/client VPN public key ##
PublicKey = 2H8vRWKCrddLf8vPwwTLMfZcRhOj10UBdc0j8W7yQAk=
 
## client VPN IP address (note  the /32 subnet) ##
AllowedIPs = 192.168.10.2/32
```

Save and close the file. Next start the service again, run:
```bash
$ sudo systemctl start wg-quick@wg0
```

## Verification

```bash
$ ping -c 4 192.168.10.1
$ sudo wg
## try to ssh into server using our VPN connection ##
$ ssh vivek@192.168.10.1
```

## Server Configuration Full Example

```bash
# ------------------------------------------------
# Config files are located in /etc/wireguard/wg0
# ------------------------------------------------

# ---------- Server Config ----------
[Interface]
Address = 10.10.0.1/24 # IPV4 CIDR 
Address = fd86:ea04:1111::1/64 # IPV6 CIDR 
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE; ip6tables -A FORWARD -i wg0 -j ACCEPT; ip6tables -t nat -A POSTROUTING -o eth0 -j MASQUERADE # Add forwarding when VPN is started
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE; ip6tables -D FORWARD -i wg0 -j ACCEPT; ip6tables -t nat -D POSTROUTING -o eth0 -j MASQUERADE # Remove forwarding when VPN is shutdown
PrivateKey = # Put server private key here
ListenPort = 51820 # Port server should be listening on

[Peer]
PublicKey = B1Oyq4HertWCcK8YBWETfoHICnFN4+tCfyouxsdhWhs= # Client public key
AllowedIPs = 10.10.0.2/32, fd86:ea04:1111::2/128 # IPs client can connect as
```
## Client Configuration Full Example

```bash
# ---------- Client Config ----------
[Interface]
Address = 10.10.0.2/32 # IPV4 address client is allowed to connect as
Address = fd86:ea04:1111::2/128 # IPV6 address client is allowed to connect as
PrivateKey = # Client private key goes here
DNS = 1.1.1.1 # DNS client should use for resolution (Cloudflare here)

[Peer]
PublicKey = WI6KwPohbGqsJUZ/FpZup2zGTaBFdeHeJCq2dtT1KBU= # Server public key
Endpoint = YOUR_SERVER:51820 # Where the server is at + the listening port
AllowedIPs = 0.0.0.0/0, ::/0 # Forward all traffic to server
```

## Important steps

```bash
# ------------------------------------------------
# Commands
# ------------------------------------------------
sudo wg-quick up wg0 # Starting wireguard
sudo wg-quick down wg0 # Shutting down wireguard

sudo wg # to see status

# ------------------------------------------------
# Watch traffic
# ------------------------------------------------
# https://nbsoftsolutions.com/blog/viewing-wireguard-traffic-with-tcpdump

# View encrypted traffic from wireless card to VPN server
sudo tcpdump -n -X -i wlp1s0 host YOUR_SERVER

# View http traffic going to tunnel
sudo tcpdump -n -v -i wg0 port 80

# ------------------------------------------------
# Misc
# ------------------------------------------------

# Start wireguard on system boot
sudo systemctl enable wg-quick@wg0

# Ensure forwarding is allowed by adding below to /etc/sysctl.conf on server
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1

# Or use this
echo "net.ipv4.ip_forward = 1
net.ipv6.conf.all.forwarding = 1" > /etc/sysctl.d/wg.conf
sysctl --system
```
