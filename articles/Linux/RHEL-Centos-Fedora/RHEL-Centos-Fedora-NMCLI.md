# RHEL Centos Fedora NMCLI


## Recreate connection

Take a not of device name for which we want to create connection


```bash
nmcli dev status
```

Now delete conection if exists

```bash
nmcli conn delete 'Wired connection 1'
```

Create new dhcp connection

```bash
nmcli con add type ethernet con-name eth0 ifname eth0
```

Activate new connection

```bash
nmcli con up eth0
```