# Spin down hdd

Umount the filesystem and then run

```bash
hdparm -S 1 /dev/sdb
```

to set it to spin down after five seconds (replace /dev/sdb with the actual device for the hard disk).
This will minimize the power used and heat generated by the hard disk.


