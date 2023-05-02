# Kubernetes网络插件


本文主要介绍k8s使用的经常使用的网络插件，以及基本原理。

<!--more-->

## flannel
### 三种网络模式
1. udp（性能很差）

2. vxlan（性能一般，常用）

3. host-gw（性能最好）
howt-gw模式的工作原理，就是将每个Flannel子网的下一跳，设置成了该子网对应的宿主机的IP地址，也就是说，宿主机（host）充当了这条容器通信路径的“网关”（Gateway），这正是host-gw的含义

