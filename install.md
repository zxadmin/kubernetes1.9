# 部署kubernetes1.9.1版本

## 准备好基础环境

两台centos7操作系统，一台作为master（192.168.1.192），一台作为node（192.168.1.193）

### 均关闭selinux，关闭firewalld，关闭swap。

1）sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/sysconfig/selinux

2）systemctl stop firewalld && systemctl disable firewalld

3）swapoff -a

永久关闭swap内存，修改/etc/fstab文件，注释掉SWAP的自动挂载，使用free -m确认swap已经关闭。


### 修改主机名和配置主机映射

master： hostnamectl set-hostname k8s-master

node：   hostnamectl set-hostname k8s-node

vim /etc/hosts

127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4

::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

192.168.1.192 k8s-master

192.168.1.193 k8s-node


## 安装软件

1）首先配置好kubernetes1.9.1 本地yum仓库

百度云链接：https://pan.baidu.com/s/1c3kUD7i 密码：8s2s

将repo目录放压在了/mnt/下，并配置好repo文件

2）vim /etc/yum.repos.d/kube.repo


[kubernetes-local-file]

name=Kubernetes offline - downloads

baseurl=file:///mnt/repo/https0x3A0x2F0x2Fpackages.cloud.google.com/yum/repos/kubernetes-el7-x86_64/

enabled=1

gpgcheck=0

3）3）安装docker

yum -y install docker && systemctl start docker && systemctl enable docker

4）安装kubernetes

yum -y install kubectl kubelet kubernetes-cni kubeadm

systemctl start kubelet && systemctl enable kubelet


## 导入镜像

镜像放在百度云链接: https://pan.baidu.com/s/1rasoi9A 密码：8i5x

1）master端镜像

gcr.io/google_containers/kube-apiserver-amd64:v1.9.1 
  
gcr.io/google_containers/kube-scheduler-amd64:v1.9.1
  
gcr.io/google_containers/kube-proxy-amd64:v1.9.1  

gcr.io/google_containers/kube-controller-manager-amd64:v1.9.1
    
quay.io/coreos/flannel:v0.9.1-amd64
   
gcr.io/google_containers/k8s-dns-sidecar-amd64:1.14.7

gcr.io/google_containers/k8s-dns-kube-dns-amd64:1.14.7

gcr.io/google_containers/k8s-dns-dnsmasq-nanny-amd64:1.14.7 

gcr.io/google_containers/etcd-amd64:3.1.10   

gcr.io/google_containers/pause-amd64:3.0

2）node端镜像

gcr.io/google_containers/kube-apiserver-amd64:v1.9.1

quay.io/coreos/flannel:v0.9.1-amd64

gcr.io/google_containers/etcd-amd64:3.1.10

gcr.io/google_containers/pause-amd64:3.0

gcr.io/google_containers/kube-proxy-amd64:v1.9.1

gcr.io/google_containers/kubernetes-dashboard-amd64:v1.9.0

docker.io/wanghkkk/heapster-influxdb-amd64-v1.3.3:v1.3.3

docker.io/wanghkkk/heapster-grafana-amd64-v4.4.3:v4.4.3

docker.io/wanghkkk/heapster-amd64-v1.4.0:v1.4.0


## kubeadm进行初始化

通过kubeadm init命令来初始化，指定一下kubernetes版本，并设置一下pod-network-cidr

kubeadm init --kubernetes-version=v1.9.1 --pod-network-cidr=10.244.0.0/16

该命令执行完后，回返回一长串提示，例如：

kubeadm join --token d506ef.63c05fa829529565 10.2.0.37:6443 --discovery-token-ca-cert-hash sha256:4cd1954bf2a1c0904f92328d33bc25471604abd918e019b3c1905289fb8130f2

这行内容先记录下，等下节点添加的时候要使用。 

如果初始化失败，可以重置下，再初始化

kubeadm reset

在执行过程中，要等几分钟,然后根据提示执行以下命令：

mkdir -p $HOME/.kube
 
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
 
chown $(id -u):$(id -g) $HOME/.kube/config


## 成功以后可以指定一下命令进行查看

kubectl get cs

NAME                 STATUS    MESSAGE              ERROR

etcd-0               Healthy   {"health": "true"}   

scheduler            Healthy   ok        
 
controller-manager   Healthy   ok


## 部署 flannel

kubectl apply -f kube-flannel.yml


## 查看各服务的状态

kubectl get pod -n kube-system

NAME                                    READY     STATUS    RESTARTS   AGE

etcd-k8s-master                         1/1       Running   2          3h

kube-apiserver-k8s-master               1/1       Running   2          3h

kube-controller-manager-k8s-master      1/1       Running   2          3h

kube-dns-6f4fd4bdf-cn7g7                3/3       Running   6          3h

kube-flannel-ds-77sk7                   1/1       Running   24         3h

kube-flannel-ds-zd5b2                   1/1       Running   2          3h

kube-proxy-7m956                        1/1       Running   0          3h

kube-proxy-qbqgx                        1/1       Running   2          3h

kube-scheduler-k8s-master               1/1       Running   2          3h

kubernetes-dashboard-5d6bd74468-8jr9x   1/1       Running   2          2h

检查下kube-dns是否安装成功。kube-dns比较重要，它负责整个集群的解析，要确保它正常运行。


## node节点部署

node节点导入镜像后执行master返回的命令

kubeadm join --token d506ef.63c05fa829529565 10.2.0.37:6443 --discovery-token-ca-cert-hash sha256:4cd1954bf2a1c0904f92328d33bc25471604abd918e019b3c1905289fb8130f2


## master：

kubectl  get  nodes

NAME         STATUS    ROLES     AGE       VERSION

k8s-master   Ready     master    3h        v1.9.0

k8s-node     Ready     <none>    3h        v1.9.0

可以看到node节点已经加入了，并且是正常的ready状态；至此，整个集群的配置完成，可以开始使用了。
