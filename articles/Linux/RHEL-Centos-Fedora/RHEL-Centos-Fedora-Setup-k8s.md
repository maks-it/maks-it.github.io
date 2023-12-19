# Fedora K8S Setup

In this guide we are setting up Kubernetes cluster based on [Production Environment](https://kubernetes.io/docs/setup/production-environment/) setup documentation.

## Cluster configuration overview

1 Router:
* rtrsrv0001.corp.maks-it.com

1 Load balancer:
* k8slbl0001.corp.maks-it.com

2 master nodes:
* k8smst0001.corp.maks-it.com
* k8smst0002.corp.maks-it.com

3 worker nodes:
* k8swrk0001.corp.maks-it.com
* k8swrk0002.corp.maks-it.com
* k8swrk0003.corp.maks-it.com

In our scenario we use router in the way to avoid Hosts file maintenance on each node. Router is set up to provide basic newtwork functionality with DNS and DHCP. 
Load balancer and all nodes must have static IP, so create MAC to IP bindings in DHCP service:

|Host name	|IP Address	| Distro|
|--|--|--|
|rtrsrv0001.corp.maks-it.com|192.168.6.1| PfSense|
|k8slbl0001.corp.maks-it.com|192.168.6.5| Fedora Server |
|k8smst0001.corp.maks-it.com|192.168.6.10| Fedora Server |
|k8smst0002.corp.maks-it.com|192.168.6.11| Fedora Server |
|k8swrk0001.corp.maks-it.com|192.168.6.20| Fedora Server |
|k8swrk0002.corp.maks-it.com|192.168.6.21| Fedora Server |
|k8swrk0003.corp.maks-it.com|192.168.6.22| Fedora Server |

## Disable Selinux

```bash
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
```

## Disable Swap

* Swap configuration. The default behavior of a kubelet was to fail to start if swap memory was detected on a node. Swap has been supported since v1.22. And since v1.28, Swap is supported for cgroup v2 only; the NodeSwap feature gate of the kubelet is beta but disabled by default.
  * You MUST disable swap if the kubelet is not properly configured to use swap. For example, sudo swapoff -a will disable swapping temporarily. To make this change persistent across reboots, make sure swap is disabled in config files like /etc/fstab, systemd.swap, depending how it was configured on your system.

```bash
swapoff -a && sed -i.bak '/\sswap\s/s/^/#/' /etc/fstab
```

On fedora you have to uninstall zram

```bash
sudo dnf remove zram-generator-defaults -y
```

## Forwarding IPv4 and letting iptables see bridged traffic 

Execute the below mentioned instructions:

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```

```bash
sudo modprobe overlay
sudo modprobe br_netfilter
```


```bash
# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
```

```bash
# Apply sysctl params without reboot
sudo sysctl --system
```

Verify that the br_netfilter, overlay modules are loaded by running the following commands:

```bash
lsmod | grep br_netfilter
lsmod | grep overlay
```

Verify that the net.bridge.bridge-nf-call-iptables, net.bridge.bridge-nf-call-ip6tables, and net.ipv4.ip_forward system variables are set to 1 in your sysctl config by running the following command:

```bash
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
```

## systemd cgroup driver

To set systemd as the cgroup driver, edit the KubeletConfiguration option of cgroupDriver and set it to systemd. For example:

```yaml
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
...
cgroupDriver: systemd
```

>Note: Starting with v1.22 and later, when creating a cluster with kubeadm, if the user does not set the cgroupDriver field under KubeletConfiguration, kubeadm defaults it to systemd.

In Kubernetes v1.28, with the KubeletCgroupDriverFromCRI feature gate enabled and a container runtime that supports the RuntimeConfig CRI RPC, the kubelet automatically detects the appropriate cgroup driver from the runtime, and ignores the cgroupDriver setting within the kubelet configuration.

## Install Docker and Docker compose

```bash
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo
```

```bash
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

```bash
sudo systemctl enable docker --now
sudo docker run hello-world
```

## Install cri-dockerd

```bash
wget https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.7/cri-dockerd-0.3.7.20231027185657.170103f2-0.fc36.x86_64.rpm
```

```bash
rpm -ivh cri-dockerd-0.3.7.20231027185657.170103f2-0.fc36.x86_64.rpm
```

```bash
systemctl enable --now cri-docker.socket
```

## Bootstrapping clusters with kubeadm

### Installing kubeadm

#### Before you begin
* A compatible Linux host. The Kubernetes project provides generic instructions for Linux distributions based on Debian and Red Hat, and those distributions without a package manager.
* 2 GB or more of RAM per machine (any less will leave little room for your apps).
* 2 CPUs or more.
* Full network connectivity between all machines in the cluster (public or private network is fine).
* Unique hostname, MAC address, and product_uuid for every node. See here for more details.
* Certain ports are open on your machines. See here for more details.

Control plane

|Protocol	|Direction	|Port Range	|Purpose	|Used By|
|--|--|--|--|--|
|TCP	|Inbound	|6443	|Kubernetes API server	|All|
|TCP	|Inbound	|2379-2380	|etcd server client API	|kube-apiserver, etcd|
|TCP	|Inbound	|10250	|Kubelet API	|Self, Control plane|
|TCP	|Inbound	|10259	|kube-scheduler	|Self|
|TCP	|Inbound	|10257	|kube-controller-manager	|Self|

Although etcd ports are included in control plane section, you can also host your own etcd cluster externally or on custom ports.

```bash
echo '<?xml version="1.0" encoding="utf-8"?>
<service>
  <port port="6443" protocol="tcp"/>
  <port port="2379-2380" protocol="tcp"/>
  <port port="10250" protocol="tcp"/>
  <port port="10259" protocol="tcp"/>
  <port port="10257" protocol="tcp"/>
</service>' | sudo tee /etc/firewalld/services/k8s-control-plane.xml > /dev/null
```


```bash
sudo firewall-cmd --zone=public --add-service=k8s-control-plane --permanent
sudo firewall-cmd --reload
sudo firewall-cmd --runtime-to-permanent
```

Worker node(s)

|Protocol	|Direction	|Port Range	|Purpose	|Used By|
|--|--|--|--|--|
|TCP	|Inbound	|10250	|Kubelet API	|Self, Control plane|
|TCP	|Inbound	|30000-32767	|NodePort Services†	|All|

† Default port range for NodePort Services.

All default port numbers can be overridden. When custom ports are used those ports need to be open instead of defaults mentioned here.

One common example is API server port that is sometimes switched to 443. Alternatively, the default port is kept as is and API server is put behind a load balancer that listens on 443 and routes the requests to API server on the default port.

```bash
echo '<?xml version="1.0" encoding="utf-8"?>
<service>
  <port port="10250" protocol="tcp"/>
  <port port="30000-32767" protocol="tcp"/>
</service>' | sudo tee /etc/firewalld/services/k8s-worker-node.xml > /dev/null
```

```bash
sudo firewall-cmd --zone=public --add-service=k8s-worker-node --permanent
sudo firewall-cmd --reload
sudo firewall-cmd --runtime-to-permanent
```


## Setting up load balancer

Update Repositories:

```bash
sudo dnf update -y
```


Install HAProxy:

Copy code

```bash
sudo dnf install haproxy -y
```

Configure HAProxy for Load Balancing Kubernetes Master Nodes:
Backup the HAProxy configuration:

bash
Copy code

```bash
sudo cp /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.backup
```

Edit HAProxy Configuration:

bash
Copy code

```bash
sudo nano /etc/haproxy/haproxy.cfg
```

Add the following configuration to the haproxy.cfg file:

plaintext
Copy code

```cfg
frontend kubernetes
    bind *:6443
    mode tcp
    option tcplog
    default_backend kubernetes-master-nodes

backend kubernetes-master-nodes
    mode tcp
    balance roundrobin
    option tcp-check
    server k8s-master-1 192.168.6.10:6443 check
    server k8s-master-2 192.168.6.11:6443 check
```

This configuration sets up a TCP frontend on port 6443 (the Kubernetes API port) and balances traffic using the round-robin algorithm between the two Kubernetes master nodes.

Save and Close the File.


```bash
sudo systemctl enable --now  haproxy
```


```bash
sudo systemctl status haproxy
```

Important Notes:
Replace 192.168.6.10 and 192.168.6.11 with the actual IP addresses of your Kubernetes master nodes.

Make sure that port 6443 is the correct port for the Kubernetes API server.

This configuration should enable HAProxy to balance traffic between your Kubernetes master nodes. However, please ensure you have a solid understanding of HAProxy and Kubernetes networking before applying these changes to a production environment. Always back up configurations and test changes in a controlled environment first.


Enable firewall rules


## Kubeadm install

```bash
# This overwrites any existing configuration in /etc/yum.repos.d/kubernetes.repo
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.28/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.28/rpm/repodata/repomd.xml.key
exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
EOF
```

```bash
sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
sudo systemctl enable --now kubelet
```

## Creating a cluster with kubeadm

(Recommended) If you have plans to upgrade this single control-plane kubeadm cluster to high availability you should specify the --control-plane-endpoint to set the shared endpoint for all control-plane nodes. Such an endpoint can be either a DNS name or an IP address of a load-balancer.

```bash
sudo kubeadm init --control-plane-endpoint 192.168.6.5 --cri-socket unix:///var/run/cri-dockerd.sock
```

```bash
sudo kubeadm reset --cri-socket unix:///var/run/cri-dockerd.sock
```

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

```bash
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Alternatively, if you are the root user, you can run:

```bash
  export KUBECONFIG=/etc/kubernetes/admin.conf
```

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Download a calico.yaml file:

```yaml
curl -O -L https://docs.projectcalico.org/manifests/calico.yaml
```

This calico.yaml file defines an IP pool for Calico to assign IPs to pods in your Kubernetes cluster. It uses VXLAN for encapsulation, enabling networking between pods across nodes in your cluster.

Then, deploy the Calico network to your Kubernetes cluster using the following command:

```yaml
kubectl apply -f calico.yaml
```


You can now join any number of control-plane nodes by copying certificate authorities
and service account keys on each node:

```
/etc/kubernetes/
│
└── pki/
    ├── ca.crt
    ├── ca.key
    ├── sa.key
    ├── front-proxy-ca.crt
    ├── front-proxy-ca.key
    ├── etcd/
    │   ├── ca.crt
    │   └── ca.key
    ├── sa.pub
```

```bash
sudo cp pki/ca.crt /etc/kubernetes/pki/ca.crt && \
sudo cp pki/ca.key /etc/kubernetes/pki/ca.key && \
sudo cp pki/sa.key /etc/kubernetes/pki/sa.key && \
sudo cp pki/front-proxy-ca.crt /etc/kubernetes/pki/front-proxy-ca.crt && \
sudo cp pki/front-proxy-ca.key /etc/kubernetes/pki/front-proxy-ca.key && \
sudo mkdir -p /etc/kubernetes/pki/etcd && \
sudo cp pki/etcd/ca.crt /etc/kubernetes/pki/etcd/ca.crt && \
sudo cp pki/etcd/ca.key /etc/kubernetes/pki/etcd/ca.key && \
sudo cp pki/sa.pub /etc/kubernetes/pki/sa.pub && \
sudo chown root:root /etc/kubernetes/pki -R
```

Run the following as root:

```bash
sudo kubeadm join 192.168.6.5:6443 --token <token> \
        --discovery-token-ca-cert-hash <discovery-token-ca-cert-hash> \
        --control-plane \
        --cri-socket unix:///var/run/cri-dockerd.sock
```

Then you can join any number of worker nodes by running the following on each as root:

```bash
sudo kubeadm join 192.168.6.5:6443 --token <token> \
        --discovery-token-ca-cert-hash <discovery-token-ca-cert-hash> \
        --cri-socket unix:///var/run/cri-dockerd.sock
```


## How to generate new join tokens

Execute the following command on the control plane node to generate a new join token:

```bash
sudo kubeadm token create --print-join-command
```

If you need to get details about the generated token, such as its value and expiration, you can use:

```bash
sudo kubeadm token list
```