# houssamdib July 2019
###############################################
## Install and Deploy Kubernetes on CentOS 7 ##
###############################################

##################
## Requirements ##
##################
    # Two macjo,es or VMs with CentOS 7 installed
    # Two static IPs configured
        # The address 192.168.35.18 is configured on the first machine which is the Master 
        # The address 192.168.35.19 is configured on the second machine which is the Worker
    # 2vCPU per machine
    # 2GB RAM per machine
    # 20GB
    # The root user or a user with root permissions
    # Swap Disabled

###################
## Initial setup ##
###################

## configure hosts and hostname on each machine, each one can communicate with each other using the hostname

## Prepare our VMs
hostnamect  set-hostname k8s-master-01 # master
hostnamect  set-hostname k8s-worker-01 # execute on worker 01
hostnamect  set-hostname k8s-worker-02 # execute on worker 02

vi /etc/hosts
192.168.35.18   k8s-master-01
192.168.35.19   k8s-worker-01
192.168.35.20   k8s-worker-02

## Disable swap
free -h # to verify swap state
swapoff -a
sed -e '/swap/s/^/#/g' -i /etc/fstab # comment line contain swap in fstab file
free -h

## Disable SELinux
# We need also to disable SELinux. On both machines.
getenforce
setenforce 0
sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config

## Network Plugin
vi /etc/sysctl.d/k8s.conf

net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1

## Install Docker
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install -y docker-ce
systemctl enable docker.service
systemctl start docker.service
systemctl status docker.service

## Install Kubernetes
vi /etc/yum.repos.d/kubernetes.repo

[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
        https://p        https://p        hum/doc/rpm-package-key.gpg

yum install -y kubelet kubeadm kubectl
systemctl enable kubelet.service
systemctl status kubelet.service


## Master Node Configuration

