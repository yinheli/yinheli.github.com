---
draft: true
date: 2020-03-11
title: Greenplum 入门
description: Greenplum 安装及基本概念入门
tags: 
  - bigdata
  - Greenplum
---

Greenplum 数据库是 shared nothing 的分析型 MPP 数据库。近期由于老婆的项目组在使用，我也顺便了解学习下。记录下主要的流程步骤备忘。

如果你参考我的文档，需要对 Greenplum 的架构有个大概的了解，官网和中文社区是个好去处。

## 环境准备

资源有限，用 docker 来模拟多个服务器的情况，先准备 docker 环境。 我这里宿主机是 centos 7， 参考 docker 官方文档, 依次安装 docker 和 docker-composer。

```bash
# setup repository
yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2

yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

yum-config-manager --disable docker-ce-nightly

yum install -y docker-ce docker-ce-cli containerd.io

# start docker
systemctl start docker

# enable docker auto start
systemctl enable docker

# install docker-composer
curl -L "https://github.com/docker/compose/releases/download/1.25.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose

# test docker and composer
docker info
docker-compose version
```

要模拟整个安装流程，因此只准备一个基础的镜像，仅安装必要的软件包。`Dockerfile` 如下：

```dockerfile
FROM centos:centos7

RUN yum install -y wget

# 够用阿里云的镜像，在国内，加速包下载
# 安装基本的软件包，并支持 sshd 服务，greenplum 节点之间通讯需要
RUN wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo && \
    yum makecache  && \
    yum install -y which bind-utils net-tools less wget curl zip unzip telnet lsof git && \
    yum install -y passwd openssh-clients openssh-server ssh-keygen && \
    ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key -N "" -q && \
    ssh-keygen -t ecdsa -f /etc/ssh/ssh_host_ecdsa_key -N "" -q && \
    ssh-keygen -t ed25519 -f /etc/ssh/ssh_host_ed25519_key -N "" -q && \
    echo 'UseDNS no' >> /etc/ssh/sshd_config

# 安装 greenplum 依赖包
RUN yum install -y ed apr apr-util bzip2 krb5-devel libevent libyaml openssl iproute

# 初始化用户和数据文件
RUN useradd -m gpadmin
RUN mkdir -p /data && chown -R gpadmin:gpadmin /data

EXPOSE 22

CMD ["/usr/sbin/sshd", "-D"]
```

build 基于 centos 7 的镜像，名为 `mygreenplum`，后面我们用到这个镜像。

```bash
docker build . -t mygreenplum
```

准备 `docker-compose.yaml`

> 规划 1 个 master 节点，2 个 segment 节点，1 个 ETL 节点（用来做 ETL 任务）
>
> 节点命名参考官方命名习惯: mdw ( master 节点 )
>
> 另外注意，需要准备一个内存大一点的服务器，我的这里的试验机为 8G 内存

```yaml
version: "3"
services:
  master:
    image: "mygreenplum"
    hostname: "mdw"
    container_name: "master"
    ports:
      - "2222:22"
      - "5432:5432"
    sysctls:
    net.core.somaxconn: 1024
    net.ipv4.tcp_syncookies: 0

    volumes:
      - .:/root/downloads
    networks:
      main:
        aliases:
          - "master"
  segment01:
    image: "mygreenplum"
    hostname: "segment01"
    container_name: "segment01"
    volumes:
      - .:/root/downloads
    networks:
      main:
        aliases:
          - "segment01"
  segment02:
    image: "mygreenplum"
    hostname: "segment02"
    container_name: "segment02"
    volumes:
      - .:/root/downloads
    networks:
      main:
        aliases:
          - "segment02"
  segment03:
    image: "mygreenplum"
    hostname: "segment03"
    container_name: "segment03"
    volumes:
      - .:/root/downloads
    networks:
      main:
        aliases:
          - "segment03"

networks:
  main:
```

github 下载 Greenplum 二进制安装包

```
wget https://github.com/greenplum-db/gpdb/releases/download/6.4.0/greenplum-db-6.4.0-rhel7-x86_64.rpm
```

启动服务

```bash
docker-composer up -d

# 查看启动情况
docker ps -a
```

## 安装

参考 [pivotal 上的安装指导](https://gpdb.docs.pivotal.io/6-4/install_guide/install_gpdb.html), 由于是在 docker 里，指导里的系统配置部分可以省略，实际生产环境建议遵循指导手册说明，以及相关参数的调优。

进入 master 节点，初始化其 gpadmin 用户 ssh 密钥

```bash
docker exec -it master /bin/bash -c 'su - gpadmin'
ssh-keygen
cat ~/.ssh/id_rsa.pub
```

进入 其他节点，把 master 节点的 ssh 公钥写入 gpadmin 用户下

```bash
docker exec -it segment01 /bin/bash -c 'su - gpadmin'
mkdir .ssh
vi .ssh/authorized_keys
chmod 700 .ssh && chmod 600 .ssh/authorized_keys
```

回到 master 节点，准备 greenplum-db 的安装

```bash
docker exec -it master /bin/bash
# root 用户执行安装
yum install -y downloads/greenplum-db-6.4.0-rhel7-x86_64.rpm
chown -R gpadmin:gpadmin /usr/local/greenplum*

# 切换到 gpadmin 修改 profile
su - gpadmin
mkdir -p /data/master
vi ~/.bash_profile
```

```bash
source /usr/local/greenplum-db/greenplum_path.sh
export MASTER_DATA_DIRECTORY=/data/master/gpseg-1
```

```bash
# 应用 profile
source ~/.bash_prifile

# 准备 hostfile_exkeys 将所有的 segment、etl 节点的 hostname 都写到里面
vi hostfile_exkeys
```

```bash
# 验证所有的节点都是可以连接的
gpssh -f hostfile_exkeys -e 'uname -a'
```

进入其他节点，和 master 节点一样，安装 greenplum-db 的 rpm 包

```bash
docker exec -it segment01 /bin/bash
yum install -y downloads/greenplum-db-6.4.0-rhel7-x86_64.rpm
chown -R gpadmin:gpadmin /usr/local/greenplum*
```

## 初始化

## 登录

## 测试导入数据

参考资料：

- Docker: https://docs.docker.com/install/
- Greenplum: https://github.com/greenplum-db/gpdb & https://greenplum.org/
- pivotal gpdp: https://gpdb.docs.pivotal.io/6-4/main/index.html
