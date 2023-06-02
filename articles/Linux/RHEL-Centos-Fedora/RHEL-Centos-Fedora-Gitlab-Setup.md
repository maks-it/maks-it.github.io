# Gitlab setup Fedora server

>Taken from:
>
>https://computingforgeeks.com/install-gitlab-ce-on-centos-fedora/
>

```bash
sudo dnf -y install curl nano policycoreutils-python3 libxcrypt-compa
```

```bash
sudo dnf -y install postfix
```

```bash
sudo systemctl enable postfix --now
```

```bash
sudo nano /etc/yum.repos.d/gitlab-ce.repo
```

```bash
[gitlab_gitlab-ce]
name=gitlab_gitlab-ce
baseurl=https://packages.gitlab.com/gitlab/gitlab-ce/el/8/$basearch
repo_gpgcheck=1
gpgcheck=1
enabled=1
gpgkey=https://packages.gitlab.com/gitlab/gitlab-ce/gpgkey
       https://packages.gitlab.com/gitlab/gitlab-ce/gpgkey/gitlab-gitlab-ce-3D645A26AB9FBD22.pub.gpg
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300

[gitlab_gitlab-ce-source]
name=gitlab_gitlab-ce-source
baseurl=https://packages.gitlab.com/gitlab/gitlab-ce/el/8/SRPMS
repo_gpgcheck=1
gpgcheck=1
enabled=1
gpgkey=https://packages.gitlab.com/gitlab/gitlab-ce/gpgkey
       https://packages.gitlab.com/gitlab/gitlab-ce/gpgkey/gitlab-gitlab-ce-3D645A26AB9FBD22.pub.gpg
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300
```

```bash
sudo dnf install -y gitlab-ce
```

```bash
sudo nano /etc/gitlab/gitlab.rb
```

```bash
external_url 'http://gitlab.example.com'
```

```bash
sudo gitlab-ctl reconfigure
```

```bash
sudo gitlab-ctl status
```

```bash
sudo firewall-cmd --permanent --add-service={ssh,http,https} --permanent
sudo firewall-cmd --reload
```

## Retrieve root password

```bash
cat /etc/gitlab/initial_root_password
```