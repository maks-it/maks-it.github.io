
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
