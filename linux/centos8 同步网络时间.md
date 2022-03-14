# centos8 同步网络时间

centos 7 还有 ntp 直接同步网络时间。但是在centos8使用了chrony去做时间同步了。本文将介绍配置国内的时间服务器，然后每次开机的时候自动同步时间。防止集群之间时间不同步。

## 修改配置文件

```shell
[root@hadoop102 atguigu]# vim /etc/chrony.conf 

pool ntp5.aliyun.com iburst

#使用 root 用户，修改上面的文件，修改成阿里云的时间服务器

```

## 设置开机启动

```shell
systemctl restart chronyd

systemctl enable chronyd
```

## 注意点

如果chrony的服务已经启动了，直接执行 chronyc tracking 命令即可同步时间。

补充：亲测这个指令不行哇 ，重启服务吧。

不是很确定：在没有启动服务的时候，可以执行下面这个同步时间。

```shell
chronyd -q 'pool ntp1.aliyun.com iburst'
```

