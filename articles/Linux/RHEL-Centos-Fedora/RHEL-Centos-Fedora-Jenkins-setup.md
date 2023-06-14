# Jenkins server setup

```bash
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
```

```bash
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
```

```bash
sudo dnf upgrade
```

## Add required dependencies for the jenkins package

```bash
sudo dnf install java-11-openjdk
```

```bash
sudo dnf install jenkins
```

```bash
sudo systemctl daemon-reload
```

```bash
sudo systemctl enable jenkins --now
```

## Firewall rules

Set env variables

```bash
YOURPORT=8080
PERM="--permanent"
SERV="$PERM --service=jenkins"
```

Configure firewalld service

```bash
sudo firewall-cmd $PERM --new-service=jenkins
sudo firewall-cmd $SERV --set-short="Jenkins ports"
sudo firewall-cmd $SERV --set-description="Jenkins port exceptions"
sudo firewall-cmd $SERV --add-port=$YOURPORT/tcp
sudo firewall-cmd $PERM --add-service=jenkins
sudo firewall-cmd --zone=public --add-service=http --permanent
sudo firewall-cmd --reload
```

## Initial admin account

```bash
nano /var/lib/jenkins/secrets/initialAdminPassword
```

## Install plugins
https://plugins.jenkins.io/gitea/
install gitea plugin
install Branch API plugin


