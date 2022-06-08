# NFS Server

```bash
sudo systemctl enable rpcbind
sudo systemctl enable nfs-server
sudo service rpcbind start
sudo service nfs-server start
```

## Creating Shares

```bash
nano /etc/exports
```

add your desired shares

```bash
/mnt/disk1  192.168.0.0/255.255.255.0(rw)
/mnt/disk2  192.168.0.0/255.255.255.0(rw)
/mnt/disk3  192.168.0.0/255.255.255.0(rw)
```

Restart the NFS service to apply the changes by typing

```bash
systemctl restart nfs-server
```

No error message should appear.

Beware: if, as superuser ("root") you overwrite by hand the "exports" file (for instance copying a file you had saved before), or if you delete the old one and create a new one by hand, then it's probable than the SEL (Security enhanced Linux) labels of the file get wrong. In such a case (or if you suspect it's the case), you have to restore them like this:

```bash
su -c 'restorecon /etc/exports'
```

The same way, you should restore the SEL labels of any configuration file you have created or restored by hand, for example /etc/hosts.allow .
