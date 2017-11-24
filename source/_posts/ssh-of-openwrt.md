---
title: ssh_of_openwrt
date: 2017-09-04 20:10:09
tags:
	- openwrt
---

openwrt 一般采用dropbear 作为ssh 客户端/服务端。

# 客户端
使用如下命令，生成public keys
``` bash
dropbearkey  -y -f /etc/dropbear/dropbear_rsa_host_key
```

将图中pulibc key 复制到服务端。
<!-- more -->
# 服务端
dropbear 与openssh 有点区别在于，**authorized\_keys 文件并不在~/.ssh/authorized_keys 文件中**，如下图：

之后重启服务端dropbear service


dropbear 的配置如下：


# 使用
在客服端使用如下命令登陆服务端
``` bash
ssh -i /etc/dropbear/dropbear_rsa_host_key root@172.28.52.151 -p 6350
```

需要使用 "-i"选项指明identify 文件， "-p 6350" 是指明端口号


或者可以不指明-i  文件
``` bash
cp /etc/dropbear/dropbear_rsa_host_key ~/.ssh/id_dropbear
```

之后使用如下命令直接登陆：
``` bash
ssh 172.28.52.151 -l root -p 6350 
```

