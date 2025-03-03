# Configuring kubeadm and deploying nginx login page

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

Before start kubeadm, we will need to configure some ports and disable swap memory

### Firewall configuration
```
sudo firewall-cmd --permanent --add-port=6443/tcp
sudo firewall-cmd --permanent --add-port=2379-2380/tcp
sudo firewall-cmd --permanent --add-port=10250/tcp
sudo firewall-cmd --permanent --add-port=10251/tcp
sudo firewall-cmd --permanent --add-port=10252/tcp
sudo firewall-cmd --permanent --add-port=10255/tcp
sudo firewall-cmd --reload
```

### Disabling swap memory
```
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
sudo swapoff -a
```

# Now we can proceed with kubeadm installation 


## Step 1: Setup containerd
```
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
```
```
modprobe overlay
modprobe br_netfilter
```
```
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
```
sysctl --system
```
apt-get install -y containerd
mkdir -p /etc/containerd
containerd config default > /etc/containerd/config.toml
```
```
nano /etc/containerd/config.toml
```
--> SystemdCgroup = true
```
systemctl restart containerd
```

## Step 2: Kernel Parameter Configuration  *I used a redhat distro*
```
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```
```
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.32/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.32/rpm/repodata/repomd.xml.key
exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
EOF
```
```
sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
```


## Step 3 Initialize Kubeadm (this will happen only on master node)
```
kubeadm init --pod-network-cidr=192.168.100.0/24 
```
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
## Step 4: Install Network Addon (flannel) (master node)
```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```


## Step 5 Creating a deploy

1. we need to create a deploy we have 2 options imperative or declarative:
*Imperative:
```
kubectl create deployment mi-app --image=nginx --replicas=3
```
*Declarative:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mi-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: mi-app
  template:
    metadata:
      labels:
        app: mi-app
    spec:
      containers:
      - name: nginx
        image: nginx
```
2. after we create the deploy we can check the status using **kubectl get pod** if status is Running we will need to create a service to expose the pod.
*Important: I chose port 30200 that is the port we will use to access that pod from our browser
```
kubectl expose deploy mi-app --name=nginx-svc --port=80 --target-port=30200 --type=NodePort 
```
