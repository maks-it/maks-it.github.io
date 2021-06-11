# Intro

## How To Install Latest Nodejs on CentOS/RHEL 7/6

For Latest Release:-

``` bash
yum install -y gcc-c++ make
curl -sL https://rpm.nodesource.com/setup_12.x | sudo -E bash -
```

For Stable Release:-

``` bash
yum install -y gcc-c++ make
curl -sL https://rpm.nodesource.com/setup_10.x | sudo -E bash -
```

``` bash
sudo yum install nodejs
```

``` bash
node -v
```