# Kubernetes Training
Kubernetes Installation and few useful commands for beginners

Kubernetes is an open-source platform that helps you deploy, scale, and manage resources across multiple containers.

# Pre-requisites
For Installation of Kubernetes make sure that you have atleast 2 machines available 1 master node and 1 slave node

# Installation Steps
Follow this tutroial and learn how to install Kubernetes on a Linux system.

First we would begin first with installation of Docker on both node i.e. on master and slave nodes

# Step 1: Docker Installation on both Master and Slave nodes:

    sudo su 
    yum install docker -y 
    systemctl enable docker && systemctl start docker

# Step 2: Configure Kubernetes Repository on both Master and Slave nodes:
	
    cat <<EOF > /etc/yum.repos.d/kubernetes.repo
    [kubernetes]
    name=Kubernetes
    baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
    enabled=1
    gpgcheck=1
    repo_gpgcheck=0
    gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
   
    EOF

# Step 3: Update Iptables Settings on both Master and Slave nodes:
Set the net.bridge.bridge-nf-call-iptables to '1' in your sysctl config file. This ensures that packets are properly processed by IP tables during filtering and port forwarding.
	
    cat <<EOF >  /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables = 1
    EOF
    sysctl --system

# Step 4: Disable SELinux on both Master and Slave nodes:
The containers need to access the host filesystem. SELinux needs to be set to disable mode, which effectively disables its security functions.

    setenforce 0

# Step 5: Disable SWAP on both Master and Slave nodes:
We need to disable SWAP to enable the kubelet to work properly:

    sudo sed -i '/swap/d' /etc/fstab
    sudo swapoff -a

# Step 6: Install kubelet, kubeadm, and kubectl on both Master and Slave nodes:
	
We would install kubelet, kubeadm and kubectl; and start kubelet daemon
Do it on both master as well as on worker nodes

    yum install -y kubelet kubeadm kubectl

    systemctl enable kubelet && systemctl start kubelet

# Step 7: Set Hostname on both Master and Slave nodes:
Give a unique hostname to each of your nodes, use this command 
### On master node
    sudo hostnamectl set-hostname master-node
### On slave node
    sudo hostnamectl set-hostname worker-node

### Make a host entry or DNS record to resolve the hostname for all nodes:
    sudo vi /etc/hosts	

	192.168.1.10 master-node
    192.168.1.20 worker-node

# Step 8: Create Cluster with kubeadm on Master nodes only:
### If Calico is used as Pod network then perform below steps on master node initialize the cluster

    sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --ignore-preflight-errors=NumCPU
    #  sudo kubeadm init --pod-network-cidr=192.168.0.0/16 #Do this only if proper CPU cores are available
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config

### If Flannel is used as Pod network then perform below steps on master node initialize the cluster

    sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --ignore-preflight-errors=NumCPU
    #  sudo kubeadm init --pod-network-cidr=10.244.0.0/16 #Do this only if proper CPU cores are available
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    

# Step 9: Join Worker Node to Cluster

On Worker nodes, Switch to the root mode
Copy kubeadm join command from output of "kubeadm init on master node" 
   
    <kubeadm join command copies from master node>

# Step 10: Set Up Pod Network
On Master Node: Clone a repository to get YAML files for Calico networking

    yum install git -y
    git clone https://github.com/parmarkunal84/kubernetes-training

On Master Node: In case if Kubernetes version is 1.21 or lower for Calico networking
		
    cd kubernetes-training/15-calico/

On Master Node: In case if Kubernetes version is 1.22 or greater for Calico networking

    cd kubernetes-training/15-calico/v1-22
      
For calico networking: Apply below commands on master node only

    sudo kubectl apply -f etcd.yaml
    sudo kubectl apply -f rbac.yaml
    sudo kubectl apply -f calico.yaml
    
On Master Node: In case if Kubernetes using Flannel as Pod networking

    kubectl apply -f https://github.com/coreos/flannel/raw/master/Documentation/kube-flannel.yml
	kubectl create -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel-rbac.yml
	
# Step 11: Check Status of Cluster

Wait for 2 minutes and see if the nodes are ready; run it on master node

    kubectl get nodes

To watch the status of system pods 

    kubectl get pods --all-namespaces


# Step 12: Set environment variables on worker nodes
Perform this step on all the worker nodes

    mkdir -p $HOME/.kube
    export KUBECONFIG=/etc/kubernetes/kubelet.conf
    
## Optional 
If you want your master to run user-defined pods as well, execute below

      kubectl taint nodes --all node-role.kubernetes.io/master-

You can also view the above Kubernetes Installation steps in 00-kubernetes-installation.md
