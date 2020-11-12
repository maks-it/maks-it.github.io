
## Prevent creation of vmmem files in VmWare

VMWare creates .vmem files to back the guest RAM. On the host this causes disk thrashing especially during powering on and off the guest.


Add the following lines to the .vmx file to prevent creation of .vmem files. This will reduce disk IO and VM performance will improve especially on non-SSD disks.

```
prefvmx.minVmMemPct = "100"
MemTrimRate = "0"
mainMem.useNamedFile = "FALSE"
sched.mem.pshare.enable = "FALSE"
prefvmx.useRecommendedLockedMemSize = "TRUE"
```