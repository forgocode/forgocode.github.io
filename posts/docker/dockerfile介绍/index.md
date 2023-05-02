# Dockerfile介绍


本文主要是记录Dockerfile的一些基本命令，便于以后构建镜像查询

<!--more-->

```
dockerfile
FROM ubuntu:14.04
MAINTAINER YS.Zou <>

ADD run /root/run
ADD sources.list /etc/apt/sources.list
ADD id_rsa.pub /tmp/pubkey
ADD requirements /root/requirements

RUN mkdir -p /root/.ssh && \
  cat /tmp/pubkey >> /root/.ssh/authorized_keys && \
  rm -rf /tmp/pubkey

CMD ["bash", "/root/run"]
```





```dockerfile
FROM：基础镜像，当前新的镜像是基于哪个已经存在的镜像的
MAINTAINER：镜像作者及其邮箱
RUN：镜像构建时需要执行的命令
EXPOSE：暴露镜像的端口号
WORKDIR：指定在创建容器后，终端默认登录进来的工作目录，一个落脚点
ENV：用来在构建镜像过程中设置环境变量
ADD：将宿主机目录下的文件拷贝进镜像且ADD命令会自动处理URL和解压tar压缩包
COPY：跟ADD类似，拷贝文件和目录到镜像中
VOLUME：容器数据卷，用于数据保存和持久化工作
CMD：指定一个容器启动时要执行的命令，dockefile可以有多个cmd指令，但只有最后一个生效，cmd会被docker run之后的参数替换
ENTRYPOINT：指定一个容器启动时要执行的命令，和CMD的目的一样，都是在指定容器启动程序及参数 --entrpoint
ONBUILD：当构建一个被继承的dockefile时运行命令，父镜像在被子继承后父镜像的onbuild被触发
```

