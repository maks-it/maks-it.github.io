
## Enable or disable IP forwarding

Using either method above will not make the change persistent. To make sure the new setting survives a reboot, you need to edit the `/etc/sysctl`.conf file.

Add one of the following lines to the bottom of the file, depending on whether you’d like Linux IP forwarding to be off or on, respectively. Then, save your changes to this file. The setting will be permanent across reboots.

```bash
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
```

After editing the file, you can run the following command to make the changes take effect right away.

```bash
sysctl -p
```

## Flush iptables

```bash
iptables --flush && \
iptables --table nat --flush && \
iptables --delete-chain && \
iptables --table nat --delete-chain
```


## Routing

```bash
PostUp = iptables -t nat -A POSTROUTING -o %i -j MASQUERADE; iptables -I FORWARD -p tcp --tcp-flags SYN,RST SYN -j TCPMSS --clamp-mss-to-pmtu
PostDown = iptables -t nat -D POSTROUTING -o %i -j MASQUERADE; iptables -D FORWARD -p tcp --tcp-flags SYN,RST SYN -j TCPMSS --clamp-mss-to-pmtu
```



