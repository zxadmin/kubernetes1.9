# CentOS 7实战部署Kubernetes1.9.1  


### 什么是Kubernetes？

Kubernetes是Google开源的容器集群管理系统，实现基于Docker构建容器，利用Kubernetes能很方便管理多台Docker主机中的容器。
  
主要功能如下：

1）将多台Docker主机抽象为一个资源，以集群方式管理容器，包括任务调度、资源管理、弹性伸缩、滚动升级等功能。

2）使用编排系统（YAML File）快速构建容器集群，提供负载均衡，解决容器直接关联及通信问题

3）自动管理和修复容器，简单说，比如创建一个集群，里面有十个容器，如果某个容器异常关闭，那么，会尝试重启或重新分配容器，始终保证会有十个容器在运行，反而杀死多余的。


### Kubernetes角色组成：

1）Pod

Pod是kubernetes的最小操作单元，一个Pod可以由一个或多个容器组成；

同一个Pod只能运行在同一个主机上，共享相同的volumes、network、namespace；

2）ReplicationController（RC）

RC用来管理Pod，一个RC可以由一个或多个Pod组成，在RC被创建后，系统会根据定义好的副本数来创建Pod数量。在运行过程中，如果Pod数量小于定义的，就会重启停止的或重新分配Pod，反之则杀死多余的。当然，也可以动态伸缩运行的Pods规模或熟悉。

RC通过label关联对应的Pods，在滚动升级中，RC采用一个一个替换要更新的整个Pods中的Pod。

3）Service

Service定义了一个Pod逻辑集合的抽象资源，Pod集合中的容器提供相同的功能。集合根据定义的Label和selector完成，当创建一个Service后，会分配一个Cluster IP，这个IP与定义的端口提供这个集合一个统一的访问接口，并且实现负载均衡。

4）Label

Label是用于区分Pod、Service、RC的key/value键值对； 

Pod、Service、RC可以有多个label，但是每个label的key只能对应一个；

主要是将Service的请求通过lable转发给后端提供服务的Pod集合；


### Kubernetes组件组成：

1）kubectl

客户端命令行工具，将接受的命令格式化后发送给kube-apiserver，作为整个系统的操作入口。

2）kube-apiserver

作为整个系统的控制入口，以REST API服务提供接口。

3）kube-controller-manager

用来执行整个系统中的后台任务，包括节点状态状况、Pod个数、Pods和Service的关联等。

4）kube-scheduler

负责节点资源管理，接受来自kube-apiserver创建Pods任务，并分配到某个节点。

5）etcd

负责节点间的服务发现和配置共享。

6）kube-proxy

运行在每个计算节点上，负责Pod网络代理。定时从etcd获取到service信息来做相应的策略。

7）kubelet

运行在每个计算节点上，作为agent，接受分配该节点的Pods任务及管理容器，周期性获取容器状态，反馈给kube-apiserver。

8）DNS

一个可选的DNS服务，用于为每个Service对象创建DNS记录，这样所有的Pod就可以通过DNS访问服务了。
