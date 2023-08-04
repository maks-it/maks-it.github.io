## Gitlab setup CentOS Stream 9


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



sudo yum install -y curl policycoreutils-python-utils openssh-server perl

## Enable OpenSSH server daemon if not enabled: sudo systemctl status sshd
sudo systemctl enable --now sshd

## Check if opening the firewall is needed with: sudo systemctl status firewalld
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo systemctl reload firewalld





sudo yum install postfix
sudo systemctl enable --now postfix




curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.rpm.sh | sudo bash


sudo EXTERNAL_URL="http://gitsrv0002.corp.maks-it.com" yum install -y gitlab-ee


Unless you provided a custom password during installation, a password will be randomly generated and stored for 24 hours in /etc/gitlab/initial_root_password . Use this password with username root to login.

cat /etc/gitlab/initial_root_password

## Enable runners

### Download the binary for your system
sudo curl -L --output /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64

### Give it permission to execute
sudo chmod +x /usr/local/bin/gitlab-runner

### Create a GitLab Runner user
sudo useradd --comment 'GitLab Runner' --create-home gitlab-runner --shell /bin/bash

### Install and run as a service
cd /usr/local/bin/
sudo ./gitlab-runner install --user=gitlab-runner --working-directory=/home/gitlab-runner
sudo gitlab-runner start


### Runner registration

> example
>
> gitlab-runner register  --url http://gitsrv0002.corp.maks-it.com  --token glrt-95wYYLaMdDxT6X6vVGmV


Hi everyone! I want to share the solution!
just add image = docker:stable  and privileged = true

/etc/gitlab-runner/config.toml