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

##  Install git

```bash
sudo dnf install git -y
```

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
```

```bash
sudo usermod -a -G docker jenkins
```

## Setup remote container registry

Probably you are using local container registry with self hosted certificate, so you have to add it to the jenkins docker host

```bash
sudo cp /home/${USER}/<yourdomain>.crt /etc/docker/certs.d/<yourdomain>/ca.crt
```


You would need to loog out and log back in so that your group membership is re-evaluated or type the following command:
su -s ${USER}
Verify that you can run docker commands without sudo.
docker run hello-world
This command downloads a test image and runs it in a container. When the container runs, it prints an informational message and exits.

If you initially ran Docker CLI commands using sudo before adding your user to the docker group, you may see the following error, which indicates that your ~/.docker/ directory was created with incorrect permissions due to the sudo commands.

WARNING: Error loading config file: /home/user/.docker/config.json -
stat /home/user/.docker/config.json: permission denied
To fix this problem, either remove the ~/.docker/ directory (it is recreated automatically, but any custom settings are lost), or change its ownership and permissions using the following commands:
sudo chown "$USER":"$USER" /home/"$USER"/.docker -R
sudo chmod g+rwx "$HOME/.docker" -R

Linux: Copy the domain.crt file to /etc/docker/certs.d/myregistrydomain.com:5000/ca.crt on every Docker host. You do not need to restart Docker.

Windows Server:

Open Windows Explorer, right-click the domain.crt file, and choose Install certificate. When prompted, select the following options:

Store location	local machine
Place all certificates in the following store	selected
Click Browser and select Trusted Root Certificate Authorities.

Click Finish. Restart Dock


## Install plugins
https://plugins.jenkins.io/gitea/
install gitea plugin
install Branch API plugin
install Pipeline: Multibranch


