"# Kubernetes-master-client" 
Agenda:- Installing and setup kubernetes master client architecture here using kubeadm.

Pre-requisite:- 
> In this setup we wil required two vm with  below specification
  2vcp, 4gb ram, atleast 8gb disk
> Both vm should have good network connectivity
  below port should be open at both node 
  6443,22,80,443

Steps:- We  will devide this steps here in two part master's steps and worker's steps.

Steps required to run on master machine.

Step 1) update and upgrade the machine then setup hostname and /etc/hosts file.
````
sudo su -
sudo apt update
sudo apt -y full-upgrade
hostnamectl set-hostname master-node
vi /etc/hosts
master-node-ip master-node
worker-node-ip worker-node
````

Step 2) Install kubelet, kubeadm and kubectl
````
sudo apt -y install curl apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

````

Step 3) Install required packages.
````
sudo apt update
sudo apt -y install vim git curl wget kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
kubectl version --client && kubeadm version         //verify installed package
````
Step 4) Disable Swap
````
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
sudo swapoff -a
sudo mount -a
free -h
````
Step 5) Enable kernel modules and configure sysctl.
````
# Enable kernel modules
sudo modprobe overlay
sudo modprobe br_netfilter

# Add some settings to sysctl
sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

# Reload sysctl
sudo sysctl --system
````
Step 6) Install Container runtime (Installing CRI-O:)
````
# Ensure you load modules
sudo modprobe overlay
sudo modprobe br_netfilter

# Set up required sysctl params
sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

# Reload sysctl
sudo sysctl --system

# Add Cri-o repo
sudo su -
OS="xUbuntu_20.04"
VERSION=1.25
echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /" > /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
echo "deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$VERSION/$OS/ /" > /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.list
curl -L https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$VERSION/$OS/Release.key | apt-key add -
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | apt-key add -

# Install CRI-O
sudo apt update
sudo apt install cri-o cri-o-runc

# Update CRI-O CIDR subnet
sudo sed -i 's/10.85.0.0/192.168.0.0/g' /etc/cni/net.d/100-crio-bridge.conf

# Start and enable Service
sudo systemctl daemon-reload
sudo systemctl restart crio
sudo systemctl enable crio
sudo systemctl status crio
````
Step 7) Initialize master node
````
lsmod | grep br_netfilter
sudo systemctl enable kubelet
sudo kubeadm config images pull --cri-socket unix:///var/run/crio/crio.sock
sudo kubeadm init \
  --pod-network-cidr=192.168.0.0/16 \
  --cri-socket unix:///var/run/crio/crio.sock \
  --upload-certs \
  --control-plane-endpoint=master-node
````
Step 8) copy join token for running on worker node.

Step 9) Setup kubeconfig folder
````
mkdir -p $HOME/.kube
sudo cp -f /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
````
----------------
Steps required to run on worker node

Step 1) update and upgrade the machine then setup hostname and /etc/hosts file.
````
sudo su -
sudo apt update
sudo apt -y full-upgrade
hostnamectl set-hostname worker-node
vi /etc/hosts
master-node-ip master-node
worker-node-ip worker-node
````
Step 2) Install kubelet, kubeadm and kubectl
````
sudo apt -y install curl apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
````
Step 3) Install required packages.
````
sudo apt update
sudo apt -y install vim git curl wget kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
kubectl version --client && kubeadm version         //verify installed package
````
Step 4) Disable Swap
````
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
sudo swapoff -a
sudo mount -a
free -h
````
Step 5) Enable kernel modules and configure sysctl.
````
# Enable kernel modules
sudo modprobe overlay
sudo modprobe br_netfilter

# Add some settings to sysctl
sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

# Reload sysctl
sudo sysctl --system

Step 6) Install Container runtime (Installing CRI-O:)
# Ensure you load modules
sudo modprobe overlay
sudo modprobe br_netfilter

# Set up required sysctl params
sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

# Reload sysctl
sudo sysctl --system

# Add Cri-o repo

OS="xUbuntu_20.04"
VERSION=1.25
echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /" > /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
echo "deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$VERSION/$OS/ /" > /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.list
curl -L https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$VERSION/$OS/Release.key | apt-key add -
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | apt-key add -

# Install CRI-O
sudo apt update
sudo apt install cri-o cri-o-runc

# Update CRI-O CIDR subnet
sudo sed -i 's/10.85.0.0/192.168.0.0/g' /etc/cni/net.d/100-crio-bridge.conf

# Start and enable Service
sudo systemctl daemon-reload
sudo systemctl restart crio
sudo systemctl enable crio
sudo systemctl status crio
````
Step 7) Enable kubelet
````
lsmod | grep br_netfilter
sudo systemctl enable kubelet
````
Step 8) Need to run join command on worker node with was generated while we ran kubeadm init command at master node.

----------------------
Run below command to verify added node in cluster
````
kubectl get nodes
````


###########Thank you###############
Please like and share this video
This installation document is available at below git link
https://github.com/LOKI0307/Kubernetes-master-client
Bye
