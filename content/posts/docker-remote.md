---
title: "docker开启远程API"
date: 2021-02-04T18:36:13+08:00
---

> docker的远程api支持tcp和ssh两种方式进行连接，ssh方式要求服务端版本大于18.09。
这里建议ssh方式，可以使用sshkey进行鉴权，使用tcp方式鉴权的话，需要生成tls证书，在无鉴权的情况下开放api非常危险，相当于裸奔。

# ssh方式

> 注意docker 服务器端版本需要大于18.09，参考：https://code.visualstudio.com/docs/containers/ssh

`docker version`查看版本

## 配置ssh key

注意，本地必须通过ssh key方式免密登录服务器，password方式不可以。

本地生成ssh key，并把公钥上传到远程服务器，访问 `ssh user@host -p port` 可以登录成功。

## 远程访问

命令中需要加入`-H ssh://[user]@[ip]:[port]`。

如`$ docker -H ssh://pi@g.233333.fun:5559 version`

# tcp方式

## 方式一：修改daemon.config

配置参考：https://blog.csdn.net/u013948858/article/details/79974796

如群晖的docker是通过配置daemon.config方式启动的，不同系统该文件可能存在位置不一样，可以通过下面的方式找到

1. 找到docker进程：`ps aux|grep docker`，返回下面内容

`root     21531  3.6  0.7 1855448 70880 ?       Ssl  Jan29 365:06 /var/packages/Docker/target/usr/bin/dockerd --config-file /var/packages/Docker/etc/dockerd.json`

2. 可以看到dockerd.json就是对应的配置文件

以下是群晖的docker配置参考：
```json
{
  "data-root": "/var/packages/Docker/target/docker",
  "hosts": [
    "unix:///var/run/docker.sock",
    "tcp://0.0.0.0:2375"
  ],
  "log-driver": "db",
  "registry-mirrors": [
    "https://mirror.tuna.tsinghua.edu.cn"
  ],
  "storage-driver": "btrfs",
  "tls": true,
  "tlscacert": "/var/services/homes/aaron/Drive/certs/ca.pem",
  "tlscert": "/var/services/homes/aaron/Drive/certs/server-cert.pem",
  "tlskey": "/var/services/homes/aaron/Drive/certs/server-key.pem",
  "tlsverify": true
}
```

其中`"tcp://0.0.0.0:2375"`表示开放tcp的2375端口进行连接，0.0.0.0接收任意ip访问，`tls=true`表示使用tls方式鉴权，下面tlscacert等等为服务端cert证书文件。

## 方式二：修改docker.service文件

修改 `sudo vi /lib/systemd/system/docker.service`

在[Service]段落中替换 `ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock`


## 生成证书

参考：https://segmentfault.com/a/1190000022023393


## 远程访问命令

在docker命令中加入`-H tcp://[ip]:[端口]`就行，其他操作与本地操作docker一样。

* 如查看容器`docker -H tcp://[ip]:2375 ps`

# docker context说明

使用context可以方便的管理不同的docker节点，[官方介绍](https://docs.docker.com/engine/context/working-with-contexts/)

通过切换不同的context，可以像操作本地容器一样，操作不同的docker节点。

常用命令有：

查看context：`$ docker context ls`

创建context：`docker context create [name] --docker "host=ssh://[user]@[ip]:[port]"`

切换context：`docker context use [name]`，默认context的为default。


# 不加sudo

默认情况下，执行docker命令是需要sudo提权的，但是ssh方式鉴权都是当前用户无法提权，直接执行就会报错找不到docker命令，所以需要给普通用户加入执行docker的权限。

* 创建docker用户组
* 把当前用户加入docker组：sudo gpasswd -a ${USER} docker
* 重启docker服务