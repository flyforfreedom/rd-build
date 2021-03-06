# docker

### docker 安装

> 官方安装文档：https://docs.docker.com/engine/installation/linux/docker-ce/centos/

中国镜像安装最新稳定版

```Shell
curl -sSL https://get.daocloud.io/docker | sh
```

键入docker -v命令，若显示如下返回则表示安装成功

```shell
[root@localhost vagrant]# docker -v
Docker version 17.03.0-ce, build 60ccb22
```

docker服务启动 重启 停止

```Shell
[root@localhost vagrant]# service docker start
Redirecting to /bin/systemctl start  docker.service

[root@localhost vagrant]# service docker restart
Redirecting to /bin/systemctl restart  docker.service

[root@localhost vagrant]# service docker stop
Redirecting to /bin/systemctl stop  docker.service
```
### docker容器网段设置

#### 分配独立IP

安装pipework

```Shell
wget https://github.com/jpetazzo/pipework/archive/master.zip
unzip master.zip
cp pipework-master/pipework  /usr/bin/
chmod +x /usr/bin/pipework
```

给宿主机创建网桥

```shell
[root@localhost ~]# vi /etc/sysconfig/network-scripts/ifcfg-eno16777736
TYPE=Ethernet
#BOOTPROTO=static
#IPADDR=172.16.146.116
#NETMASK=255.255.255.0
#GATEWAY=172.16.146.1
#DNS1=8.8.8.8
#DNS2=192.168.252.3
BOOTPROTO="none"
DEFROUTE="yes"
PEERDNS="yes"
PEERROUTES="yes"
NAME=eno16777736
ONBOOT=yes
BRIDGE="br0"
~
```

设置物理网卡桥接到网桥

```shell
[root@localhost ~]# vi /etc/sysconfig/network-scripts/ifcfg-br0
TYPE="Bridge"
BOOTPROTO=static
IPADDR=172.16.146.116
NETMASK=255.255.255.0
GATEWAY=172.16.146.1
PREFIX=24
DNS1=172.16.2.2
DNS2=114.114.114.114
NAME=br0
ONBOOT=yes
DEVICE=br0
```

重启网络  

```shell
service network restart
```

给容器分配一个独立IP的例子  

```shell
docker run -d -it --net=none --name test01 centos
pipework br0 test01 172.16.2.3/24@172.16.2.1
```

### docker镜像仓库

#### 本地仓库搭建

* 使用本地gitlab仓库（优先）

  参看软件服务[gitlab](/build/gitlab.md)章节

* 搭建本地仓库（不建议）

  ```
  docker run -d -p 5000:5000 --restart=always --name registry \
    -v /data/registry:/var/lib/registry \
    registry:2
  ```

#### 在私有仓库上传、下载、搜索镜像

> https://yeasy.gitbooks.io/docker_practice/content/repository/local_repo.html

commit镜像

当我们运行一个容器的时候（如果不使用卷的话），我们做的任何文件修改都会被记录于容器存储层里。而 Docker 提供了一个 docker commit 命令，可以将容器的存储层保存下来成为镜像。换句话说，就是在原有镜像的基础上，再叠加上容器的存储层，并构成新的镜像。以后我们运行这个新镜像的时候，就会拥有原有容器最后的文件变化。

`docker commit` 的语法格式为：

    docker commit [选项] <容器ID或容器名> [<仓库名>[:<标签>]]

    $ docker commit \
        --author "lvyalin" \
        --message "修改了默认网页" \
        webserver \
        nginx:v2
    sha256:07e33465974800ce65751acc279adc6ed2dc5ed4e0838f8b86f0c87aa1795214

其中 --author 是指定修改的作者，而 --message 则是记录本次修改的内容。这点和 git 版本控制相似，不过这里这些信息可以省略留空。

dockerfile镜像

>https://yeasy.gitbooks.io/docker_practice/content/image/dockerfile/
