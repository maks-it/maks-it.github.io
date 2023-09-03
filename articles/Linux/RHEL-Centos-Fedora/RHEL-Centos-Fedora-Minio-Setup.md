# Minio Setup

## Deploy MinIO: Single-Node Multi-Drive

```bash
mkfs.xfs /dev/sdb -L DISK1
mkfs.xfs /dev/sdc -L DISK2
mkfs.xfs /dev/sdd -L DISK3
mkfs.xfs /dev/sde -L DISK4
```

```bash
nano /etc/fstab
```

then add following lines

```bash
# <file system>  <mount point>  <type>  <options>         <dump>  <pass>
LABEL=DISK1      /mnt/disk1     xfs     defaults,noatime  0       2
LABEL=DISK2      /mnt/disk2     xfs     defaults,noatime  0       2
LABEL=DISK3      /mnt/disk3     xfs     defaults,noatime  0       2
LABEL=DISK4      /mnt/disk4     xfs     defaults,noatime  0       2
```

## Deploy Single-Node Multi-Drive MinIO

### 1) Download the MinIO Server

```bash
wget https://dl.min.io/server/minio/release/linux-amd64/archive/minio-20230831153116.0.0.x86_64.rpm -O minio.rpm
```
```bash
sudo dnf install minio.rpm
```

```bash
groupadd -r minio-user
```

```bash
useradd -M -r -g minio-user minio-user
```

```bash
chown minio-user:minio-user /mnt/disk1 /mnt/disk2 /mnt/disk3 /mnt/disk4
```

### 3) Create the Environment Variable File

```bash
nano /etc/default/minio
```

```bash
# MINIO_ROOT_USER and MINIO_ROOT_PASSWORD sets the root account for the MinIO server.
# This user has unrestricted permissions to perform S3 and administrative API operations on any resource in the deployment.
# Omit to use the default values 'minioadmin:minioadmin'.
# MinIO recommends setting non-default values as a best practice, regardless of environment.

MINIO_ROOT_USER=myminioadmin
MINIO_ROOT_PASSWORD=minio-secret-key-change-me

# MINIO_VOLUMES sets the storage volumes or paths to use for the MinIO server.
# The specified path uses MinIO expansion notation to denote a sequential series of drives between 1 and 4, inclusive.
# All drives or paths included in the expanded drive list must exist *and* be empty or freshly formatted for MinIO to start successfully.

MINIO_VOLUMES="/mnt/disk{1...4}"

# MINIO_SERVER_URL sets the hostname of the local machine for use with the MinIO Server.
# MinIO assumes your network control plane can correctly resolve this hostname to the local machine.

# Uncomment the following line and replace the value with the correct hostname for the local machine.

#MINIO_SERVER_URL="http://minio.example.net"

MINIO_OPTS=--console-address ":9001"
```

### 4) Start the MinIO Service

```bash
sudo systemctl enable --now minio
```

```bash
sudo systemctl status minio
```

```bash
journalctl -f -u minio
```

The journalctl output should resemble the following:

```bash
Status:         1 Online, 0 Offline.
API: http://192.168.2.100:9000  http://127.0.0.1:9000
RootUser: myminioadmin
RootPass: minio-secret-key-change-me
Console: http://192.168.2.100:9090 http://127.0.0.1:9090
RootUser: myminioadmin
RootPass: minio-secret-key-change-me

Command-line: https://min.io/docs/minio/linux/reference/minio-mc.html
   $ mc alias set myminio http://10.0.2.100:9000 myminioadmin minio-secret-key-change-me

Documentation: https://min.io/docs/minio/linux/index.html
```

### 5) Connect to the MinIO Service

You can access the MinIO Console by entering any of the hostnames or IP addresses from the MinIO server Console block in your preferred browser, such as http://localhost:9000

Log in with the `MINIO_ROOT_USER` and `MINIO_ROOT_PASSWORD` configured in the environment file specified to the container.