# Installation and Configuration

After os(centos7) installation, static ips were set for all vms.
But there was a minor issue, when running ifconfig command. It was throwing an error that "ifconfig command not found".
To solve this:
>Addede namervers(8.8.8.8, 8.4.4.4) on /etc/resolv.conf
>Changed the parameter "ONBOOT=no" to "ONBOOT=yes" on /etc/sysconfig/netwrork-scripts/ifcfg-xxxxx
>service network restart
>yum install net-tools
Issue was resolved, initially unable to run the command"yum install net-tools" as well.

Step1: Preparing the instances to intall Kubernetes.

changed static hostnames as follows and saved on the location /etc/hosts on all vms.
192.168.11.51	kmaster
192.168.11.52	kworker1
192.168.11.53	kworker2

Step2: Updating OS: yum update -y

Step 3: Disable SElinux 
By disabling the SElinux, all containers can easily access the host filesystem.

We can Disable SElinux by two methods

Run the below command
setenforce 0
sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
 

Go to SELinux configuration file and disable it
nano /etc/sysconfig/selinux and type SELINUX=disabled

Step 4: Disable or Off the SWAP
By disabling the SWAP, kubelet will work perfectly.

Run the below command to disable SWAP:
swapoff -a && sed -i '/swap/d' /etc/fstab


Step 5: To Allow Ports in Firewall or Disable Firewall
By allowing the below ports or disabling firewall, all containers, network drivers, and pods are communicating across the kubernetes cluster properly

Run the following command to allow ports in firewall:
firewall-cmd --permanent --add-port=6443/tcp
firewall-cmd --permanent --add-port=2379-2380/tcp
firewall-cmd --permanent --add-port=10250/tcp
firewall-cmd --permanent --add-port=10251/tcp
firewall-cmd --permanent --add-port=10252/tcp
firewall-cmd --permanent --add-port=10255/tcp
firewall-cmd –-reload




Run the below command to disable firewall (This step is not recommended for production environment, but in this article, we are going to do disable firewall)

systemctl stop firewalld
systemctl disable firewalld



Step 6: To update the IP Tables run the following command
By updating IP Tables, Port forwarding and Filtering process will work perfectly

Run the below command to update the IP tables:
modprobe br_netfilter
echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables



Step 7: To Install Docker and Kubernetes in nodes, need to configure docker and Kubernetes repositories
 

yum install wget
wget -O /etc/yum.repos.d/epel-rhsm.repo http://repos.fedorapeople.org/repos/candlepin/subscription-manager/epel-subscription-manager.repo
yum install subscription-manager -y

#Docker:- Run the below command to add docker repo

yum install yum-utils #to use config manager
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo



#Kubernetes: - Run the below command to add Kubernetes repo


nano /etc/yum.repos.d/kubernetes.repo


Paste the below details in nano editor

[kubernetes]
name=Kubernetes

baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64

enabled=1

gpgcheck=1

repo_gpgcheck=1

gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg

	https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg


Step 8: To install the docker and Kubernetes components
Run the following command to install the Kubenetes/Docker (kublet kubeadm kubectl docker)

yum install kubelet kubeadm kubectl docker -y

If it throw an error, please remove below lines from /etc/yum.repos.d/kubernetes.repo
"
gpgcheck=1
repo_gpgcheck=1
"
These verify package signatures after download.




Step 9: To start and enable Kubernetes and docker services
Run the below commands to start:

systemctl start docker && systemctl enable docker

systemctl start kubelet && systemctl enable kubelet


Step 10: To run cluster configuration in Master node, this step should follow only in master node
Run the below command to start cluster configuration in master node



kubeadm init --apiserver-advertise-address=192.168.11.51 --ignore-preflight-errors all --pod-network-cidr=10.244.0.0/16 --token-ttl 0

output:
Your Kubernetes control-plane has initialized successfully!

TTo start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.11.51:6443 --token t18fkb.mmz7th84r3hcjjhm \
    --discovery-token-ca-cert-hash sha256:ba3bb270d70c219a61db843084d0cc37d7862f5266a45033dc9ae70c3bef2468




Step 11: To check the all pods are running successful in cluster 
Run the command you can see all pods in namespaces

kubectl get pods --all-namespaces



You can see the coredns service not yet started, still in pending, So that we need to install flannel network plugin to run coredns to start pod network communication.

Step 12: To Install Flannel Pod network driver
Run the below command to install POD network
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml


Now you can see coredns and all pods in namespaces are in ready and running successfully.

Step 13: To Taint master node as a Master 
Run the below command to taint the master node and make as a master.

kubectl taint nodes --all node-role.kubernetes.io/master-


 

Step 14: Join the Worker Nodes to Master Node.
Run the token which produced by master node in other nodes to join to the cluster. 

Generated Token:-

kubeadm join 192.168.11.51:6443 --token ulnfdt.xvtfyfem2zc284xl \
    --discovery-token-ca-cert-hash sha256:fc87d035fec2a45214219a0e648cb1aceb5aecd81c13ca31cb7bfa3f6ae1ce35




Run the command to check all the nodes are connected to cluster or not

Kubectl get nodes


If anything goes wrong with with cluster cpnfiiguration. It can be erased/reset by running below command on all instances.

kubeadm reset.

Then start the cluster configuration again.
