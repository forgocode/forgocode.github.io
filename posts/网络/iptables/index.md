# Iptables


本文主要介绍了iptables的相关知识

<!--more-->

## iptables
iptables+netfilter实现了防火墙的功能，其中netfilter是一个包过滤的内核框架，iptables可以认为是一个命令行工具实现对netfilter功能的使用

## 基本组成
主要是四表五链，四表是raw，mangle，filter，nat。五链是PREROUTING, FORWARD, POSTROUTING, INPUT, OUTPUT
### 表-table
1. raw<!--  -->

2. mangle

3. filter

4. nat


### 链-chain

1. PERROUTINE

2. POSTROUTING

3. FORWARD

4. INPUT

5. OUTPUT


## 流量走向

1. 进入本机的流量
> PREROUTING -> INPUT -> 应用程序

2. 从本机到外面的流量
> 应用程序 -> OUTPUT -> POSTROUTING


3. 通过本机转发的流量
> PREROUTING -> FORWARD ->POSTROUTING

## 匹配原则

先链后表，匹配顺序如下:

![](/images/image/2023-05-02iptables01.png)

## 相关参数

### 匹配条件 
### 执行动作 -j
1. ACCEPT
2. DROP
3. REJECT
4. SNAT
5. DNAT
6. MASQUERADE
7. REDIRECT
8. LOG
9. NFQUEUE
10. RETURN

## 相关命令

