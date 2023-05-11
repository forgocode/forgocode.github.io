# Kubernetes介绍（一）


本文主要对k8s进行一个简单的基本介绍，对其相关的组件和基本概念有一个大概的了解

<!--more-->

## Kubernetes 是什么
Kubernetes 是一个可移植的、可扩展的开源平台，用于管理容器化的工作负载和服务，可促进声明式配置和自动化。简单来说，k8s就是一个管理工具，管理的对象就是容器。其中，在Master 节点上运行着集群管理相关的一组进程。这些进程实现了整个集群的资源管理、Pod调度、弹性伸缩、安全控制、系统监控和纠错等管理功能，并且都是全自动完成的。Node作为集群中的工作节点，运行真正的应用程序，在Node上Kubernetes管理的最小运行单元是Pod。Node上运行着Kubernetes的kubelet、kube-proxy服务进程，这些服务进程负责Pod的创建、启动、监控、重启、销毁，以及实现软件模式的负载均衡器。


## 三个命令行工具
### kubeadm

>* `kubeadm init`这个命令初始化集群，如果在这里出现了错误的话在命令后面加上--v=5，后面的数字越大，表示打印出来的信息等级就越高
>`kubeadm init --pod-network-cidr=192.169.1.0/16`
>这段命令的意思就是初始化一个集群，所有在这个集群上创建pod的IP都属于192.168.0.0/16这个IP段内，其中master这个节点上的IP范围是192.168.1.0/16。
>* `kubeadm join`这个命令是把其他主机添加到master这个集群里。后面要跟的参数就是前面通过`kubeadm init`生成的令牌
>* `kubeadm reset` 还原 `kubeadm init` 或者 `kubeadm join` 对主机所做的任何更改
>* `kubeadm token`管理`kubeadm init`生成的令牌，可以查看，也可以重新生成，因为这个令牌是每隔24原来的令牌就会失效一次，如果此时想要重新添加主机到这个集群里面，就必须重新生成令牌`kubeadm token create/list`
```
`kubeadm token create/list`查看口令token
#查看后面的哈希值
  -kubeadm token create --print-join-command 这个命令会直接生成一个新的命令
  -kubeadm token create 生成新的token
  -openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin-outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //' 算出新的哈希值，其实都一样
#查看各个节点的状态
kubectl get node
```
执行`kubeadm init`之后会得到的三个命令

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
这个就是kubectl的配置文件，

### kubectl

这个命令就是k8s为我们提供的命令行工具
>* apply/create 创建资源
>> kuebctl apply -f pod.yml
>* delete 删除资源
>> kubectl delete -f pod.yml
>* get/describe 查看资源
>
>> kubectl get pod -o wide -n namesapce /-A

>> kubectl describe pod (pod_name)查看资源的详细信息
>> * api-resources  查看所有的api
>> kubectl api-resources 
>* exec 进入某个容器
>
>> kubectl exec -it pod_name -n namespace -c containers bash/sh


### kubelet

kubelet 是在每个 Node 节点上运行的主要 “节点代理”。他是基于 PodSpec 来工作的。每个 PodSpec 是一个描述 Pod 的 YAML 或 JSON 对象。 kubelet 接收通过各种机制（主要是通过 apiserver）提供的一组 PodSpec，并确保这些 PodSpec 中描述的容器处于运行状态且运行状况良好。 kubelet 不管理不是由 Kubernetes 创建的容器。主要有以下几个功能：

1. pod 管理：kubelet 定期从所监听的数据源获取节点上 pod/container 的期望状态（运行什么容器、运行的副本数量、网络或者存储如何配置等等），并调用对应的容器平台接口达到这个状态。
2. 容器健康检查：kubelet 创建了容器之后还要查看容器是否正常运行，如果容器运行出错，就要根据 pod 设置的重启策略进行处理。
3. 容器监控：kubelet 会监控所在节点的资源使用情况，并定时向 master 报告，资源使用数据都是通过 cAdvisor 获取的。知道整个集群所有节点的资源情况，对于 pod 的调度和正常运行至关重要。

## 安装命令(ubuntu22.04)
```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl

sudo curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl

kubeadm config images pull

配置容器运行时
crictl config runtime-endpoint unix:///var/run/containerd/containerd.sock

# 下载crictl 

# etcd一直重启
vim /etc/containerd/config.toml

[plugins."io.containerd.grpc.v1.cri".containerd.runtimes]
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
          runtime_type = "io.containerd.runc.v2"
          runtime_engine = ""
          runtime_root = ""
          privileged_without_host_devices = false
          base_runtime_spec = ""
          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
            SystemdCgroup = true


http://10.182.34.112:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/login

kubectl proxy --address 0.0.0.0 --accept-hosts '.*'


```

## 基础概念
从大到小，有两个角色，一个是master节点，另一个是worker节点。
![image](https://img.imgdb.cn/item/6040895b360785be5445c631.png)
### master

上面有各种组件

![image](https://img.imgdb.cn/item/6040895b360785be5445c621.png)

#### apiserver
>
>> k8s API Server提供了k8s各类资源对象（pod,RC,Service等）的增删改查及watch等HTTP Rest接口，是整个系统的数据总线和数据中心。提供其他模块之间的数据交互和通信的枢纽（其他模块通过API Server查询或修改数据，只有API Server才直接操作etcd）

>>* kubelet 与 apiserver交互

>> 每个Node节点上的kubelet定期就会调用API Server的REST接口报告自身状态，API Server接收这些信息后，将节点状态信息更新到etcd中。kubelet也通过API Server的Watch接口监听Pod信息，从而对Node机器上的POD进行管理。

>>* kube-controller-manager与API Server交互

>> kube-controller-manager中的Node Controller模块通过API Server提供的Watch接口，实时监控Node的信息，并做相应处理。

>>*  kube-scheduler与API Server交互

>> Scheduler通过API Server的Watch接口监听到新建Pod副本的信息后，它会检索所有符合该Pod要求的Node列表，开始执行Pod调度逻辑。调度成功后将Pod绑定到目标节点上。

> 特别说明: 为了缓解各模块对API Server的访问压力，各功能模块都采用缓存机制来缓存数据，各功能模块定时从API Server获取指定的资源对象信息（LIST/WATCH方法），然后将信息保存到本地缓存，功能模块在某些情况下不直接访问API Server，而是通过访问缓存数据来间接访问API Server。

![image](https://img.imgdb.cn/item/60408e2f360785be54484210.png)

#### scheduler

>* 功能概览
>调度 是指将 Pod 放置到合适的 Node 上，然后对应 Node 上的 Kubelet 才能够运行这些 pod。调度器通过 kubernetes 的监测（Watch）机制来发现集群中新创建且尚未被调度到 Node 上的 Pod。 调度器会将发现的每一个未调度的 Pod 调度到一个合适的 Node 上来运行。 调度器会依据下文的调度原则来做出调度选择。

> 对每一个新创建的 Pod 或者是未被调度的 Pod，kube-scheduler 会选择一个最优的 Node 去运行这个 Pod。然而，Pod 内的每一个容器对资源都有不同的需求，而且 Pod 本身也有不同的资源需求。因此，Pod 在被调度到 Node 上之前， 根据这些特定的资源调度需求，需要对集群中的 Node 进行一次过滤。

> 在一个集群中，满足一个 Pod 调度请求的所有 Node 称之为 可调度节点。 如果没有任何一个 Node 能满足 Pod 的资源请求，那么这个 Pod 将一直停留在 未调度状态直到调度器能够找到合适的 Node。

> 调度器先在集群中找到一个 Pod 的所有可调度节点，然后根据一系列函数对这些可调度节点打分， 选出其中得分最高的 Node 来运行 Pod。之后，调度器将这个调度决定通知给 kube-apiserver，这个过程叫做 绑定。

> 在做调度决定时需要考虑的因素包括：单独和整体的资源请求、硬件/软件/策略限制、 亲和以及反亲和要求、数据局域性、负载间的干扰等等。

>*  调度流程
>1. 过滤  找到所有可供选择的节点 （可以进行信息设置，以及配置优先级）(预选算法、可供选择)
>2. 打分  确定资源发布到得分最高的节点上面（优选算法）

#### controller-manager

> 在manager内部有一组控制器节点控制器（Node Controller），副本控制器（Replication Controller），端点控制器（Endpoints Controller），服务帐户和令牌控制器（Service Account & Token Controllers）。从逻辑上讲，每个控制器都是一个单独的进程， 但是为了降低复杂性，它们都被编译到同一个可执行文件，并在一个进程中运行。每一种控制器会追踪不同类型的资源，每一种资源对象都有一个spec的字段 。这个字段说明了在这个集群中该项资源的预期状态，而控制器就是负责该项资源的当前状态尽可能接近期望状态。

> Job 是一种 Kubernetes 资源，它运行一个或者多个 Pod， 来执行一个任务然后停止。在集群中，当 Job 控制器拿到新任务时，它会保证一组 Node 节点上的 kubelet 可以运行正确数量的 Pod 来完成工作。 Job 控制器不会自己运行任何的 Pod 或者容器。Job 控制器是通知 API 服务器来创建或者移除 Pod。 控制面中的其它组件 根据新的消息作出反应（调度（scheduler）并运行新 Pod（kubelet））并且最终完成工作。 

#### kube proxy 

>* 在每个节点上运行
>* 代理 UDP、TCP 和 SCTP
>* 不支持 HTTP
>* 提供负载均衡能力
>* 只用来访问 Service

#### apiserver proxy

>* 是一个建立在 apiserver 内部的“堡垒”
>* 将集群外部的用户与群集 IP 相连接，这些IP是无法通过其他方式访问的
>* 运行在 apiserver 进程内
>* 客户端到代理使用 HTTPS 协议 (如果配置 apiserver 使用 HTTP 协议，则使用 HTTP 协议)
>* 通过可用信息进行选择，代理到目的地可能使用 HTTP 或 HTTPS 协议
>* 可以用来访问 Node、 Pod 或 Service
>* 当用来访问 Service 时，会进行负载均衡



## 其他概念

### pod

#### pod创建流程

![image](https://img.imgdb.cn/item/6045b8edcef1ec5e6f4f586f.png)

>1.用户通过kubectl命名发起请求  

>2.apiserver通过对应的kubeconfig进行认证，认证通过后将yaml中的pod信息存到etcd。

>3.Controller-Manager通过apiserver的watch接口发现了pod信息的更新，执行该资源所依赖的拓扑结构整合，整合后将对应的信息写到etcd，此时pod已经可以被调度了。

>4.Scheduler同样通过apiserver的watch接口更新到pod可以被调度，通过算法给pod分配节点，并将pod和对应节点绑定的信息写到etcd，然后将pod交给kubelet。

>5.kubelet收到pod后，调用CNI接口给pod创建pod网络，调用CRI接口去启动容器，调用CSI进行存储卷的挂载。

>6.网络，容器，存储创建完成后pod创建完成，等业务进程启动后，pod运行成功。

### service
分布式集群架构的核心：Service(服务)

一个Service对象有如下关键特征：

- 一个唯一指定的名字
- 一个虚拟IP和端口号
- 能够提供某种远程服务能力
- 被映射到了提供这种服务能力的一组容器应用

服务进程基于socket进程对外提供服务，虽然一个Service 通常由多个相关的服务进程来提供服务，每个服务进程都有一个独立的Endpoint（IP+Port）访问点，但Kubernetes 能够让我们通过Service（虚拟Cluster IP+Service Port）连接到指定的Service上（NAT）?

### 。。。




## 相关操作命令

create -f name.yaml，将创建的yaml发布到集群中 位置

get rc 查看创建的RC文件

get pods 查看pod的情况

docker ps查看正在运行的容器

get service 查看服务

get node 查看集群有多少个Node

kubectl describe node <name> 查看某个Node的详细信息

删除节点/服务 kubectl delete node/service name

kubectl describe node node1 查看详细信息

```Bash
# 查看现在证书的过期时间
kubeadm alpha certs check-expiration
# 更新证书
kubeadm alpha certs renew all

# 修改配置
cd ~/.kube
$ mv config conf.archive.2021
# Copy the new configuration file created using kubeadm
$ cp /etc/kubernetes/admin.conf config
# apply permissions to your current admin user and group
$ sudo chown $(id -u):$(id -g) config
```


