# Harbor Setup

## Firewall rules

open 80 and 443

## Install Docker and Docker compose

```bash
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo
```

```bash
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

```bash
sudo systemctl enable docker --now
sudo docker run hello-world
```

## Install Harbor

> Taken from:
>
> https://goharbor.io/docs/2.8.0/install-config/download-installer/
>
> https://goharbor.io/docs/2.8.0/install-config/configure-yml-file/
>
> https://goharbor.io/docs/2.8.0/install-config/configure-https/

```bash
wget https://github.com/goharbor/harbor/releases/download/v2.8.1/harbor-offline-installer-v2.8.1.tgz
```

```bash
tar xzvf harbor-offline-installer-v2.8.1.tgz
```

### Create SSL certificates

```bash
sudo mkdir /data/cert
cd /data/cert
```

```bash
openssl genrsa -out ca.key 4096
```

```bash
openssl req -x509 -new -nodes -sha512 -days 3650 \
 -subj "/C=IT/ST=MN/L=Ostiglia/O=maks-it/OU=dev/CN=hcrsrv0001.corp.maks-it.com" \
 -key ca.key \
 -out ca.crt
```

```bash
openssl genrsa -out hcrsrv0001.corp.maks-it.com.key 4096
```

```bash
openssl req -sha512 -new \
 -subj "/C=IT/ST=MN/L=Ostiglia/O=maks-it/OU=dev/CN=hcrsrv0001.corp.maks-it.com" \
 -key hcrsrv0001.corp.maks-it.com.key \
 -out hcrsrv0001.corp.maks-it.com.csr
```

```bash
cat > v3.ext <<-EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1=hcrsrv0001.corp.maks-it.com
DNS.2=hcrsrv0001.corp.maks-it
DNS.3=hcrsrv0001
EOF
```

```bash
openssl x509 -req -sha512 -days 3650 \
 -extfile v3.ext \
 -CA ca.crt -CAkey ca.key -CAcreateserial \
 -in hcrsrv0001.corp.maks-it.com.csr \
 -out hcrsrv0001.corp.maks-it.com.crt
```

### Create harbor configuration file

```bash
cd ~/harbor
```

```bash
cp harbor.yml.tmpl harbor.yml
```

#### Specify SSL cert and key place
```bash
/data/cert/hcrsrv0001.corp.maks-it.com.crt
/data/cert/hcrsrv0001.corp.maks-it.com.key
```

#### Start install process

```bash
sudo ./install.sh
```