# Centos8安装Nginx教程



## 安装指令

```java
yum install -y nginx 
```

安装之前需要配置阿里云的镜像源，这样下载就会快很多。

## 验证

浏览器访问本机的ip地址，出现nginx的页面就可以。

## 配置文件

/etc/nginx/nginx.conf

这是配置文件的路径，如果改了端口，要检查这个端口是不是已经放开了。

## 常用命令

重启服务：service nginx reload

查看状态：systemctl status nginx

