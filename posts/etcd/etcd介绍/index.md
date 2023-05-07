# Etcd介绍


本文主要介绍一下etcd，简单了解一下ectd的作用和使用场景

<!--more-->

## ETCD 介绍
在k8s集群中，ETCD是一个重要的组件。它主要的作用就是用来同步信息，同步配置。在k8s集群中主要有以下信息需要来同步：
> 1. 同步k8s本身的元数据信息，各个资源的状态dengdeng
> 2. 同时k8s中的网络插件，例如flannel，calico等网路插件的配置信息也需要保存在etcd中

## 简单原理
etcd是一个内存型的KV数据库。主要通过raft一致性算法实现，主要用于共享配置和服务发现。
> TODO: 详细原理介绍

## 使用etcd存储kubernetes对象信息
在ECTD中存储的kubernetes数据都保存在`/registry/`路径下，etcdctl get --prefix /registry/

## 通过ETCD 存储flannel的网络信息
在安装flannel的时候配置FLANNEL_ETCD_PREFIX= ""，这是flannel查询etcd的地址，通过以下命令来查询flannel的网络信息
```
etcdctl --ca-file=/etc/kubernetes/ssl/ca.pem --cert-file=/etc/kubernetes/ssl/kubernetes.pem --key-file=/etc/kubernetes/ssl/kubernetes-key.pem ls /kube-centos/network -r
```
通过以下命令查看指定配置
```
etcdctl --ca-file=/etc/kubernetes/ssl/ca.pem --cert-file=/etc/kubernetes/ssl/kubernetes.pem --key-file=/etc/kubernetes/ssl/kubernetes-key.pem get /kube-centos/network/config
```

## Etcd V2 与 V3 版本 API 的区别
Etcd V2 和 V3 之间的数据结构完全不同，互不兼容，也就是说使用 V2 版本的 API 创建的数据只能使用 V2 的 API 访问，V3 的版本的 API 创建的数据只能使用 V3 的 API 访问。这就造成我们访问 etcd 中保存的 flannel 的数据需要使用 etcdctl 的 V2 版本的客户端，而访问 kubernetes 的数据需要设置 ETCDCTL_API=3 环境变量来指定 V3 版本的 API。

