---
title: "docker开启远程API"
date: 2021-02-04T18:36:13+08:00
---

# 生成证书

参考：https://segmentfault.com/a/1190000022023393

# tcp修改配置开启远程端口

## 方式一：修改daemon.config

配置参考：https://blog.csdn.net/u013948858/article/details/79974796

```
{"data-root":"/var/packages/Docker/target/docker","hosts":["unix:///var/run/docker.sock","tcp://0.0.0.0:2375"],"log-driver":"db","registry-mirrors":["https://mirror.tuna.tsinghua.edu.cn"],"storage-driver":"btrfs","tls":true,"tlscacert":"/var/services/homes/aaron/Drive/certs/ca.pem","tlscert":"/var/services/homes/aaron/Drive/certs/server-cert.pem","tlskey":"/var/services/homes/aaron/Drive/certs/server-key.pem","tlsverify":true}
```

## 方式二：

修改 `sudo vi /lib/systemd/system/docker.service`
加入 `ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock`
