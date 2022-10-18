# 前言
Docker自身的4种网络工作方式和自定义网络模式

| 网络模式   |	简介  |
| --------- |  --------------------------------------------|
| Host	    | 容器将不会虚拟出自己的网卡，配置自己的IP等，而是使用宿主机的IP和端口。 |
| Bridge	| 此模式会为每一个容器分配、设置IP等，并将容器连接到一个docker0虚拟网桥，通过docker0网桥以及Iptables nat表配置与宿主机通信。|
| None	    | 该模式关闭了容器的网络功能。 |
| Container	| 创建的容器不会创建自己的网卡，配置自己的IP，而是和一个指定的容器共享IP、端口范围。 |
| 自定义网络	| 略 |

### 一、默认网络
安装Docker时，它会自动创建网络。
```shell
[root@localhost ~]# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
b95dfa0e690f   bridge    bridge    local
f93d9ee0ecdd   host      host      local
3b54b1c6233b   none      null      local
```
### 二、使用网络参数
```shell
--net=bridge : 默认选项，用网桥的方式来连接docker容器。
--net=host : docker跳过配置容器的独立网络栈。
--net=container:NAME_or_ID : 告诉docker让这个新建的容器使用已有容器的网络配置。
--net=none : 告诉docker为新建的容器建立一个网络栈，但不对这个网络栈进行任何配置，所以只能访问本地网络，没有外网
```
1. 建立网络空间
    ```shell
    #默认 DRIVER：bridge
    [root@localhost ~]# docker network create elastic
    [root@localhost ~]# docker network ls
    NETWORK ID     NAME      DRIVER    SCOPE
    b95dfa0e690f   bridge    bridge    local
    060c876e8bc8   elastic   bridge    local
    f93d9ee0ecdd   host      host      local
    3b54b1c6233b   none      null      local
    ```
2. 指定 **elastic：bridge** 网络空间运行容器
    ```shell
    [root@localhost ~]# docker run --name es01 --net elastic -d dockerwkm/ubuntu:latest
    [root@localhost ~]# docker run --name es02 --net elastic -d dockerwkm/ubuntu:latest
    
    #主机中不能访问容器中网络
    [root@localhost ~]# ping es01
    ping: es01: 未知的名称或服务
    
    # elastic网络空间中的容器可以通过 name 访问
    [root@localhost ~]# docker exec -it es02 /bin/bash
    root@5e7016eb9ab2:/bin# ping es01
    PING es01 (172.18.0.2): 56 data bytes
    64 bytes from 172.18.0.2: icmp_seq=0 ttl=64 time=0.052 ms
    64 bytes from 172.18.0.2: icmp_seq=1 ttl=64 time=0.084 ms
    64 bytes from 172.18.0.2: icmp_seq=2 ttl=64 time=0.134 ms
   
    # 容器运行时，直接指定网络到 elastic
    [root@localhost ~]# docker network connect elastic es01
    ```

3. 使用 **host** 模式运行容器(仅适用于 Linux 主机https://docs.docker.com/network/host/)
   ```shell
   #使用宿主机的网络启动
   [root@localhost ~]# docker run --name=nginx --net=host -d nginx:stable-alpine-perl
   #访问测试
   [root@localhost ~]# curl http://localhost
   ```

## 参考资料
- https://blog.csdn.net/meltsnow/article/details/94490994
- https://www.cnblogs.com/purpleraintear/p/6012695.html