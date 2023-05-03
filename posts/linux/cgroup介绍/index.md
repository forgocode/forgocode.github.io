# Cgroup介绍


本文主要介绍了cgroup的概念，以及机制

<!--more-->

## cgroup介绍
cgroup的全称是是control groups，控制组。是内核提供的对物理资源隔离的机制，通过这种机制，就可以对单个进程或者一组进程进行资源隔离/限制。比如：可以通过cgroup限制指定进程可以使用的cpu和内存数量，超过时可以选择暂停或者杀掉，这个也是docker和k8s实现的基础。

### 基本概念

1. 任务。在cgroup中，任务就是进程
2. 控制组。控制组指明了该控制组下面所有进程的资源配额。进程可以加入到某个控制组，也可以迁移到另外一个控制组
3. 层级。 控制组可以多层嵌套，子控制组继承父控制组的属性
4. 子系统 资源监视器。比如memory子系统可以控制进程内存的使用，子系统需要加入到某个层级，然后该层级的所有控制组都受到这个子系统的控制

* 子进程自动加入父进程的控制组。但是可以迁移
* 两个层级在有且只有一个子系统的时候，可以共享
* 一个层级可以有多个子系统
* 一个任务可以是多个控制组的成员，但这些控制组只能在不同的层级


### 子系统
>
>cpu: 限制进程的 cpu 使用率。  
>cpuacct 子系统，可以统计 cgroups 中的进程的 cpu 使用报告。  
>cpuset: 为cgroups中的进程分配单独的cpu节点或者内存节点。  
>memory: 限制进程的memory使用量。  
>blkio: 限制进程的块设备io。  
>devices: 控制进程能够访问某些设备。  
>net_cls: 标记cgroups中进程的网络数据包，然后可以使用tc模块（traffic control）对数据包进行控制。  
>net_prio: 限制进程网络流量的优先级。  
>huge_tlb: 限制HugeTLB的使用。  
>freezer:挂起或者恢复cgroups中的进程。  
>ns: 控制cgroups中的进程使用不同的namespace。  

### 使用
内核将cgroup的功能和配置入口通过虚拟文件系统暴露给用户态， 目录在`/sys/fs/cgroup`下面

#### cpu子系统
>1. cpu.shares: cgroup对时间的分配。比如cgroup A设置的是1，cgroup B设置的是2，那么B中的任务获取cpu的时间，是A中任务的2倍。
>2. cpu.cfs_period_us: 完全公平调度器的调整时间配额的周期。
>3. cpu.cfs_quota_us: 完全公平调度器的周期当中可以占用的时间。
>4. cpu.stat 统计值
> * nr_periods 进入周期的次数
> * nr_throttled 运行时间被调整的次数
> * throttled_time 用于调整的时间

#### cpyacct子系统(只是cpu资源报告，不做资源限制)
>1. uacct.usage: 该cgroup中所有任务总共使用的CPU时间（ns纳秒）
>2. cpuacct.stat: 该cgroup中所有任务总共使用的CPU时间，区分user和system时间。
>3. cpuacct.usage_percpu: 该cgroup中所有任务使用各个CPU核数的时间。

#### cpuset子系统
>1. cpuset.cpus: 可以使用的cpu节点
>2. cpuset.mems: 可以使用的mem节点
>3. cpuset.memory_migrate: 内存节点改变是否要迁移？
>4. cpuset.cpu_exclusive: 此cgroup里的任务是否独享cpu？
>5. cpuset.mem_exclusive： 此cgroup里的任务是否独享mem节点？
>6. cpuset.mem_hardwall: 限制内核内存分配的节点（mems是用户态的分配）
>7. cpuset.memory_pressure: 计算换页的压力。
>8. cpuset.memory_spread_page: 将page cache分配到各个节点中，而不是当前内存节点。
>9. cpuset.memory_spread_slab: 将slab对象(inode和dentry)分散到节点中。
>10. cpuset.sched_load_balance: 打开cpu set中的cpu的负载均衡。
>11. cpuset.sched_relax_domain_level: the searching range when migrating tasks
>12. cpuset.memory_pressure_enabled: 是否需要计算 memory_pressure?

#### memory
>memory.usage_in_bytes # 当前内存中的使用量  
>memory.memsw.usage_in_bytes # 当前内存和交换空间中的使用量  
>memory.limit_in_bytes # 设置or查看内存使用量  
>memory.memsw.limit_in_bytes # 设置or查看 内存加交换空间使用量  
>memory.failcnt # 查看内存使用量被限制的次数  
>memory.memsw.failcnt # - 查看内存和交换空间使用量被限制的次数  
>memory.max_usage_in_bytes # 查看内存最大使用量  
>memory.memsw.max_usage_in_bytes # 查看最大内存和交换空间使用量  
>memory.soft_limit_in_bytes # 设置or查看内存的soft limit  
>memory.stat # 统计信息  
>memory.use_hierarchy # 设置or查看层级统计的功能  
>memory.force_empty # 触发强制page回收  
>memory.pressure_level # 设置内存压力通知  
>memory.swappiness # 设置or查看vmscan swappiness 参数  
>memory.move_charge_at_immigrate # 设置or查看 controls of moving charges?  
>memory.oom_control # 设置or查看内存超限控制信息(OOM killer)  
>memory.numa_stat # 每个numa节点的内存使用数量  
>memory.kmem.limit_in_bytes # 设置or查看 内核内存限制的硬限  
>memory.kmem.usage_in_bytes # 读取当前内核内存的分配  
>memory.kmem.failcnt # 读取当前内核内存分配受限的次数  
>memory.kmem.max_usage_in_bytes # 读取最大内核内存使用量  
>memory.kmem.tcp.limit_in_bytes # 设置tcp 缓存内存的hard limit  
>memory.kmem.tcp.usage_in_bytes # 读取tcp 缓存内存的使用量  
>memory.kmem.tcp.failcnt # tcp 缓存内存分配的受限次数  
>memory.kmem.tcp.max_usage_in_bytes # tcp 缓存内存的最大使用量  
