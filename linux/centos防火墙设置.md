# centos防火墙设置

CentOS7以前的版本默认使用`iptables服务`进行管理防火墙规则。但是CentOS7以及以上版本默认使用`firewall`管理防火墙。所以在CentOS8中，就使用其默认的`firewalld`配置防火墙。

## 常见的firewall指令

```
systemctl start firewalld.service    # 启动防火墙

systemctl stop firewalld.service     # 关闭防火墙

systemctl status firewalld.service   # 查看防火墙状态

systemctl enable firewalld.service   # 设置防火墙随着系统启动（一般用来设置开机自启动）

systemctl disable firewalld.service  # 禁止防火墙随着系统启动

firewall-cmd --state           # 查看防火墙状态

firewall-cmd --reload          # 更新防火墙规
firewall-cmd --list-ports      # 查看所有打开的端口

```

## 端口相关的指令

```
firewall-cmd --query-port=8080/tcp # 查询端口是否开放

firewall-cmd --add-port=80/tcp --permanent #永久添加80端口例外(全局)

firewall-cmd --remove-port=80/tcp --permanent #永久删除80端口例外(全局)

firewall-cmd --reload #重启防火墙(修改配置后要重启防火墙)
```

