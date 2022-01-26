# Deploy KMS server with vlmcsd

KMS activates computers on a local network, eliminating the need for individual computers to connect to Microsoft. To do this, KMS uses a client–server topology. KMS client computers can locate KMS host computers by DNS or static configration. when server qualify for KMS activation, it can provide the activation service.

With this principle many people developed KMS activetools, but some of them works not very well. It is even worth to see Malicious code in these tools. vlmcsd provide a KMS server simulator write with C language, which under the GNU license. It is also multi platform, so it can runing in Windows, linux, MacOS.

# Deploy KMS service
1. choose compatible binary according to system version and CPU type. For example, my OS is CentOS 6.5 and the hardware platform is KVM visual machine(intel CPU), I should choose vlmcsd-x64-musl-static in \binaries\Linux\intel\static.
2. create a new folder ‘kms’ in /use/local.
```
mkdir -p /usr/local/kms
```
3. if binary is uploaded from local machine, it should have execute permission.
```
chmod u+x /usr/local/kms/vlmcsd-x64-musl-static
````
4. run the program
```
/usr/local/kms/vlmcsd-x64-musl-static
```
5. if you use firewall, you should open a port for this program
```
firewall-cmd --permanent --add-port=1688/tcp
````

# Test
You can use vlmcs to test
```
vlmcs-Windows-x64.exe -v address of kms server
```

# Activate Windows
Start a Command Prompt as an Administrator, then type following command:
```
slmgr /skms address of kms server
slmgr /ipk xxxxx-xxxxx-xxxxx-xxxxx-xxxxx
slmgr /ato
slmgr /xpr
```

the kms client install key can be fond in [Official Documents](https://docs.microsoft.com/en-us/windows-server/get-started/kmsclientkeys).