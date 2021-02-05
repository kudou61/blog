---
title: 使用SSH公钥登陆
date: 2016-04-27T10:09:34+08:00
---

SSH 是一台 Linux 主机的标准配置。
简单来说，SSH 是一种网络协议，用于计算机之间的加密登陆。[more](http://www.ruanyifeng.com/blog/2011/12/ssh_remote_login.html)
如果一个用户从本地计算机，使用 SSH 协议登陆到另一台计算机，我们可以认为，这种登录是安全的。


# 基本用法

SSH 主要用户远程登录，如果你要用用户名 user，登陆远程主机 host，只需要输入下面的命令即可。

```bash
$ ssh user@host
```

SSH 的默认端口是 22，你也可以使用 p 参数来指定端口登陆。

```bash
$ ssh user@host -p 22
```

发送登陆请求后，会要求用户输入密码，如果密码正确，就可以登陆。

# 公钥登陆

如果使用密码登陆，每次都要输入密码，十分麻烦。不过 SSH 还提供了公钥登陆，可以省去输入密码的步骤。
所谓"公钥登录"，原理很简单，就是用户将自己的公钥储存在远程主机上。登录的时候，远程主机会向用户发送一段随机字符串，用户用自己的私钥加密后，再发回来。远程主机用事先储存的公钥进行解密，如果成功，就证明用户是可信的，直接允许登录 shell，不再要求密码。
这种方法要求用户提供自己的公钥，如果没有，可以使用 ssh-keygen 生成。

```bash
$ ssh-keygen
```

默认是使用 RSA 算法加密的。可以使用 t 参数来修改加密算法。

```bash
$ ssh-keygen -t dsa
```

上面的命令表示使用 dsa 算法加密
运行时会提示对私钥设置口令，这里不进行设置，直接回车就行。
结束后，会在\$HOME/.ssh 目录下生成两个文件：id_rsa.pub 和 id_rsa。前者是你的公钥，后者是你的私钥。
这时再输入下面的命令可以将公钥发送到远程主机 host 上。

```bash
$ ssh user@host 'mkdir -p .ssh && cat >> .ssh/authorized_keys' < ~/.ssh/id_rsa.pub
```

运行时会要求输入 user 用户的密码，认证通过后，会在 host 的\$HOME/.ssh 目录下创建 authorized_keys 文件，用户的公钥就保存在这个文件中。
传送完毕后，可以再登陆一次，看看是否还需要密码。

# .ssh权限

如果登录一直失败，可以检查下authorized_keys文件权限是否644

```bash
➜  ~ ll .ssh
total 20K
-rw-r--r--  1 aaron users  407 Dec 30 14:57 authorized_keys
-rw-r--r--  1 aaron users   26 Jan 26 19:31 environment
-rw-------  1 aaron users  411 Dec 23 17:59 id_ed25519
-rwxrwxrwx+ 1 aaron users  101 Dec 23 17:54 id_ed25519.pub
-rwxrwxrwx+ 1 aaron users 1.8K Jan 26 14:50 known_hosts
```

# ssh登录配置

在`/etc/ssh/sshd_config`文件有相关配置

```
# 是否启用公钥登录
PubkeyAuthentication yes
# 是否启用密码验证登录
PasswordAuthentication yes
```