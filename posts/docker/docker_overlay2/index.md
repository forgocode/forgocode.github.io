# Docker_overlay2


本文主要介绍了docker的UnionFS（联合文件系统）的解决方案-overlay2

<!--more-->

## 发展历程
docker在最开始的时候使用的是AUFS和DeviceMapper作为联合文件系统的驱动。在内核3.18中，linux引入了overlay，Docker此时开始使用overlay作为UnionFS的解决方案。但由于第一代的overlay存在很多弊端，比如在运行一段时间之后，docker会报too many links perblem的错误，所以在内核4.0的时候对overlay做了很多必要的改进。此时的overlayFS 被称为overlay2.

## 使用overlay2的条件
### Docker版本高于17.06.02

### Linux版本
如果是Centos/RHEL，linux内核必须高于3.10.0-54。其他发行版本内核必须高于4.0

## overlay2 工作原理
overlay2和AUFS类似，他把所有的目录都称为层（layer），overlay2的目录是容器和进行镜像分层的基础，把这些层统一展现到一层的过程叫做联合挂载（union mount）。overlay2把目录的下一层称为lowerdir，上一层叫做upperdir，联合挂载后的目录叫做merged。
> 简单来讲，overlay2在每一层只存储改动的文件，记录上下层之间的关系，构造镜像/容器的时候，执行联合挂载，在容器中看到完整的系统文件  
>overlay2文件系统最多支持128层文件挂载

## overlay2如何读取，修改，删除文件
主要用到了写时复制技术（copy-on-write）
### 读取文件
读取文件的时候先从容器层（可读写层）查找，如果没找到，就从镜像层（只读层）复制一份到容器层。
### 修改文件
直接修改容器层中复制过来的文件
### 删除文件
删除容器层的文件，然后将只读层对应的文件置为不可访问，并没有真正删除该文件

