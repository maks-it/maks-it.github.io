# AlmaLinux K8S Setup

In this guide we are setting up Kubernetes cluster based on [Production Environment](https://kubernetes.io/docs/setup/production-environment/) setup documentation.

## Cluster configuration overview

1 Router:
* rtrsrv0001.corp.maks-it.com

1 NFS server:
* nfssrv0001.corp.maks-it.com

1 Load balancer:
* k8slbl0001.corp.maks-it.com

3 master nodes:
* k8smst0001.corp.maks-it.com
* k8smst0002.corp.maks-it.com
* k8smst0003.corp.maks-it.com

3 worker nodes:
* k8swrk0001.corp.maks-it.com
* k8swrk0002.corp.maks-it.com
* k8swrk0003.corp.maks-it.com

In our scenario we use router in the way to avoid Hosts file maintenance on each node. Router is set up to provide basic newtwork functionality with DNS and DHCP. 
Load balancer and all k8s nodes must have static IP, so create MAC to IP bindings in DHCP service.

On nfs server personally I have 2 pools:

|Pool name|Raid type|Mount point|Size|
|--|--|--|--|
|pool-1|raid1|/storage/pool-1|320Gb|
|pool-2|raid0|/storage/pool-2|1.5Tb|

`pool-1` will be used as storage for kubernetes cluster.

|Host name|IP Address|Distro|CPUs|RAM|HDD Size|
|--|--|--|
|rtrsrv0001.corp.maks-it.com|192.168.6.1|PfSense|1|2Gb|20Gb|
|nfssrv0001.corp.maks-it.com|DHCP Lease |AlmaLinux|1|2Gb|20Gb|
|k8slbl0001.corp.maks-it.com|192.168.6.5|AlmaLinux|2|2Gb|20Gb|
|k8smst0001.corp.maks-it.com|192.168.6.10|AlmaLinux|2|4Gb|200Gb|
|k8smst0002.corp.maks-it.com|192.168.6.11|AlmaLinux|2|4Gb|200Gb|
|k8swrk0001.corp.maks-it.com|192.168.6.20|AlmaLinux|2|4Gb|200Gb|
|k8swrk0002.corp.maks-it.com|192.168.6.21|AlmaLinux|2|4Gb|200Gb|
|k8swrk0003.corp.maks-it.com|192.168.6.22|AlmaLinux|2|4Gb|200Gb|

## Disable Selinux

On each `k8s*` node disable selinux

```bash
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
```

## Disable Swap

On each `k8s*` node disable swap, or perform partitioning without it during system install

* Swap configuration. The default behavior of a kubelet was to fail to start if swap memory was detected on a node. Swap has been supported since v1.22. And since v1.28, Swap is supported for cgroup v2 only; the NodeSwap feature gate of the kubelet is beta but disabled by default.
  * You MUST disable swap if the kubelet is not properly configured to use swap. For example, sudo swapoff -a will disable swapping temporarily. To make this change persistent across reboots, make sure swap is disabled in config files like /etc/fstab, systemd.swap, depending how it was configured on your system.

```bash
swapoff -a && sed -i.bak '/\sswap\s/s/^/#/' /etc/fstab
```

In case of fedora server you have to uninstall zram

```bash
sudo dnf remove zram-generator-defaults -y
```

In My case I used AlmaLinux with regular partitioning without swap and home dir

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
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

```bash
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

```bash
sudo systemctl enable docker --now
sudo docker run hello-world
```

# Change cgroup Driver

```bash
echo '{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}' | sudo tee /etc/docker/daemon.json > /dev/null
```

```bash
sudo systemctl daemon-reload
```

```bash
sudo systemctl restart docker
```

## Install cri-dockerd

On Fedora server

```bash
wget https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.7/cri-dockerd-0.3.7.20231027185657.170103f2-0.fc36.x86_64.rpm
```

```bash
rpm -ivh cri-dockerd-0.3.7.20231027185657.170103f2-0.fc36.x86_64.rpm
```

```bash
systemctl enable --now cri-docker.socket
```


On AlmaLinux

```bash
wget https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.8/cri-dockerd-0.3.8.amd64.tgz
```

```bash
tar xvf cri-dockerd-0.3.8.amd64.tgz
```

```bash
sudo mv cri-dockerd/cri-dockerd /usr/local/bin/
```

```bash
cri-dockerd --version
```

```bash
wget https://raw.githubusercontent.com/Mirantis/cri-dockerd/master/packaging/systemd/cri-docker.service https://raw.githubusercontent.com/Mirantis/cri-dockerd/master/packaging/systemd/cri-docker.socket
```

```bash
sudo mv cri-docker.socket cri-docker.service /etc/systemd/system/
```

```bash
sudo sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable cri-docker.service
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

```bash
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
    server k8s-master-3 192.168.6.12:6443 check
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


## Setting up NFS Server

```bash
sudo sysemctl enable --now cockpit
```

Begin by installing the NFS service by running the following command from a terminal window:

```bash
sudo dnf install rpcbind nfs-utils -y
```

Open firewall ports

```bash
firewall-cmd --zone=public --permanent --add-service=mountd
firewall-cmd --zone=public --permanent --add-service=nfs
firewall-cmd --zone=public --permanent --add-service=rpc-bind
firewall-cmd --reload 
```

Next, configure these services so that they automatically start at boot time:

```bash
systemctl enable rpcbind nfs-server
```

Once the services have been enabled, start them as follows:

```bash
sudo systemctl start rpcbind nfs-server
```

Add Cockpit plugins to easely manage your shares

```bash
# dnf or yum
sudo dnf install https://github.com/45Drives/cockpit-identities/releases/download/v0.1.12/cockpit-identities-0.1.12-1.el8.noarch.rpm
```

```bash
# dnf or yum
sudo dnf install https://github.com/45Drives/cockpit-file-sharing/releases/download/v3.2.9/cockpit-file-sharing-3.2.9-2.el8.noarch.rpm
```

By default root account is not able to write any on nfs volumes, but as k8s is running as root we need to disable this hardening

```bash
exportfs -v
```

```bash
/storage/pool-1
                <world>(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)
/storage/pool-2
                <world>(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)
/storage/pool-1/dapr
                <world>(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)
```

If you see `root_squash`, edit:

```bash
nano /etc/exports.d/cockpit-file-sharing.exports
```

Then add `no_root_squash` to each line, apply changes and restart services:

```bash
exportfs -ra
```

```bash
systemctl restart nfs-server
```

Verify changes:

```bash
exportfs -v
```

```bash
/storage/pool-1
                <world>(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,no_root_squash,no_all_squash)
/storage/pool-2
                <world>(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,no_root_squash,no_all_squash)
/storage/pool-1/dapr
                <world>(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,no_root_squash,no_all_squash)
```

## Kubeadm install

Production environment:

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

Learning environment:

```bash
# This overwrites any existing configuration in /etc/yum.repos.d/kubernetes.repo
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.29/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.29/rpm/repodata/repomd.xml.key
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
sudo mkdir /etc/kubernetes/pki && \
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


## Kubectl install

```bash
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
```

Validate the kubectl binary against the checksum file:
```bash
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
```

If valid, the output is:

```bash
kubectl: OK
```

If the check fails, sha256 exits with nonzero status and prints output similar to:

```bash
kubectl: FAILED
sha256sum: WARNING: 1 computed checksum did NOT match
```

>Note: Download the same version of the binary and checksum.

Install kubectl

```bash
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

Copy .kube folder from your server user account to your workstation

## Pod Network Configuration

Configure a pod network to enable the master node to schedule pods.

### Calico

Download and deploy calico.yaml file:

```yaml
curl -O -L https://docs.projectcalico.org/manifests/calico.yaml
```

This calico.yaml file defines an IP pool for Calico to assign IPs to pods in your Kubernetes cluster. It uses VXLAN for encapsulation, enabling networking between pods across nodes in your cluster.

Then, deploy the Calico network to your Kubernetes cluster using the following command:

```yaml
kubectl apply -f calico.yaml
```

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

Check the status of the node:

```bash
kubectl get nodes
```

The master node now shows the Ready status.

```bash
NAME                          STATUS   ROLES           AGE   VERSION
k8smst0001.corp.maks-it.com   Ready    control-plane   9h    v1.29.0
k8smst0002.corp.maks-it.com   Ready    control-plane   9h    v1.29.0
k8smst0003.corp.maks-it.com   Ready    control-plane   9h    v1.29.0
k8swrk0001.corp.maks-it.com   Ready    <none>          9h    v1.29.0
k8swrk0002.corp.maks-it.com   Ready    <none>          9h    v1.29.0
k8swrk0003.corp.maks-it.com   Ready    <none>          9h    v1.29.0
```

## Appendix

### How force to remove node from cluster

```bash
kubectl drain <node-name> --ignore-daemonsets --force --delete-local-data
```

```bash
kubectl cordon <node-name>
```

```bash
kubectl delete node <node-name>
```

### How to generate new join tokens

Execute the following command on the control plane node to generate a new join token:

```bash
sudo kubeadm token create --print-join-command
```

If you need to get details about the generated token, such as its value and expiration, you can use:

```bash
sudo kubeadm token list
```

### How to list all pods' ports within a namespace

```bash
kubectl get pods --namespace argocd -o=json | jq '.items[] | {name: .metadata.name, containerPorts: .spec.containers[].ports[]?.containerPort}'
```

### Setup local folder provisioning

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: dapr-storage-class
  namespace: dapr-system  # Assigning StorageClass to dapr-system namespace
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```


First, create a PersistentVolume that represents the local node folder:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: dapr-storage-pv
  namespace: dapr-system  # Assigning PersistentVolume to dapr-system namespace
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /storage/pool-1/dapr  # Path on the node where the local storage is mounted
    type: DirectoryOrCreate  # You can use DirectoryOrCreate or Directory
```

Then, create a PersistentVolumeClaim that references this PersistentVolume:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dapr-storage-pvc
  namespace: dapr-system  # Assigning PersistentVolumeClaim to dapr-system namespace
spec:
  storageClassName: "dapr-storage-class"
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

Once you've created both the PersistentVolume and PersistentVolumeClaim, you can use the PVC in your pod's configuration by referencing the claim name under persistentVolumeClaim in the pod spec.

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: test-pod
spec:
  containers:
  - name: test-pod
    image: busybox:latest
    command: ["/bin/sh"]
    args: ["-c", "touch /mnt/data/SUCCESS && sleep 600"]
    volumeMounts:
      - mountPath: "/mnt/data"
        name: dapr-storage
  restartPolicy: "Never"
  volumes:
    - name: dapr-storage
      persistentVolumeClaim:
        claimName: dapr-storage-pvc
```

### Login to pod bash

```bash
kubectl exec --stdin --tty my-release-rabbitmq-0 -n rabbitmq-system -- /bin/bash
```