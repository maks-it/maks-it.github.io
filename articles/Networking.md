# Networking

## Create network bridge

You can use the `bridge` object ip the `ip` command, or the `bridge` command that makes part of the iproute2 package.

### Basic link manipulation
To create a bridge named `br0`, that have `eth0` and `eth1` as members:

```bash
ip link add name br0 type bridge
ip link set dev br0 up
ip link set dev eth0 master br0
ip link set dev eth1 master br0
```

To remove an interface from the bridge:

```bash
ip link set dev eth0 nomaster
```

And finally, to destroy a bridge after no interface is member:

```bash
ip link del br0
```

### Forwarding manipulation
To manipulate other aspects of the bridge like the FDB[Forwarding Database](http://www.embeddedlinux.org.cn/linux_net/0596002556/understandlni-CHP-16-SECT-13.html) I suggest you to take a look at the `bridge(8)` [command](http://man7.org/linux/man-pages/man8/bridge.8.html). Examples:

Show forwarding database on `br0`

```bash
bridge fdb show dev br0
```

Disable a port(`eth0`) from processing [BPDUs](https://en.wikipedia.org/wiki/Bridge_Protocol_Data_Unit). This will make the interface filter any incoming bpdu

```bash
bridge link set dev eth0 guard on
```

Setting [STP Cost](http://packetlife.net/blog/2008/sep/5/spanning-tree-port-costs/) to a port(`eth1` for example):

```bash
bridge link set dev eth1 cost 4
```

To set root guard on eth1:

```bash
bridge link set dev eth1 root_block on
```

Cost is calculated using some factors, and the link speed is one of them. Using a fix cost and disabling the processing of BPDUs and enabling root_block is somehow simmilar to a `guard-root` feature from switches.

Other features like vepa, veb and [hairpin](https://lwn.net/Articles/347344/) mode can be found on `bridge link` sub-command list.

### VLAN rules manipulation
The `vlan` object from the bridge command will allow you to create ingress/egress filters on bridges.

To show if there is any vlan ingress/egress filters:

```bash
bridge vlan show
```

To add rules to a given interface:

```bash
bridge vlan add dev eth1 <vid, pvid, untagged, self, master>
```

To remove rules. Use the same parameters as vlan add at the end of the command to delete a specific rule.

```bash
bridge vlan delete dev eth1
```

#### Related stuff:

* [bridge(8)](http://man7.org/linux/man-pages/man8/bridge.8.html) manpage
* [How to create a bridge interface](http://baturin.org/docs/iproute2/#Create%20a%20bridge%20interface)