# kubernetes 1.15

kubernetes安装部署

yum install -y bash-com* ipvsadm
source /etc/profile

https://packages.cloud.google.com/yum/pool/548a0dcd865c16a50980420ddfa5fbccb8b59621179798e6dc905c9bf8af3b34-kubernetes-cni-0.7.5-0.x86_64.rpm
https://packages.cloud.google.com/yum/pool/14bfe6e75a9efc8eca3f638eb22c7e2ce759c67f95b43b16fae4ebabde1549f3-cri-tools-1.13.0-0.x86_64.rpm
https://packages.cloud.google.com/yum/pool/548a0dcd865c16a50980420ddfa5fbccb8b59621179798e6dc905c9bf8af3b34-kubernetes-cni-0.7.5-0.x86_64.rpm

安装docker
    yum install yum-utils device-mapper-persistent-data lvm2
    yum-config-manager   --add-repo   https://download.docker.com/linux/centos/docker-ce.repo
    yum update && yum install docker-ce-18.06.2.ce
    systemctl status docker
    修改docker cgroups
    vim /etc/docker/daemon.josn
    {
    "exec-opts": ["native.cgroupdriver=systemd"]
    }

    systemctl restart docker

    docker info | grep Cgroup
    Cgroup Driver: systemd




开放端口：
    master：
        firewall-cmd --add-port=6443/tcp --permanent
        firewall-cmd --add-port=2379-2380/tcp --permanent 
        firewall-cmd --add-port=10250-10252/tcp --permanent
    node：
        firewall-cmd --add-port=10250/tcp --permanent
        firewall-cmd --add-port=30000-32767/tcp --permanent



预安装：
yum localinstall -y cri-tools-1.13.0-0.x86_64.rpm  kubeadm-1.15.3-0.x86_64.rpm  kubectl-1.15.3-0.x86_64.rpm  kubelet-1.15.3-0.x86_64.rpm  kubernetes-cni-0.7.5-0.x86_64.rpm --disableexcludes=kubernetes

selinux：
setenforce 0

内核参数设定：
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

modprobe br_netfilter
sysctl -p /etc/sysctl.d/k8s.conf




国内kubernetes源
cat <<EOF > /etc/yum.repos.d/kubernetes.repo

[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg

EOF


Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.44.163:6443 --token hoyqzh.6si8kcqzo3b7mn88 --discovery-token-ca-cert-hash sha256:da7e9a9bc2ff5eea7078092485150691007955da8e22595013aac3b152311d04 