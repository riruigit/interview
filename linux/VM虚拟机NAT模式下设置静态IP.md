# VM虚拟机NAT模式下设置静态IP

VMware为我们提供了三种网络工作模式，分别为：桥接模式（Bridged）、网络地址转换模式（NAT）、仅主机模式（Host-only）。我在实训过程中老师教我们配置仅主机模式，但是在使用过程中，仅主机模式不能上网！！！这是一个大问题啊。但是如果直接使用NAT模式，IP会变，这又是一个烦人的问题。

## 仅主机模式介绍

在 Host-only 模式下，虚拟网络是一个全封闭的网络，它唯一能够访问的就是主机，当然多个虚拟机之间也可以互相访问。其实 Host-only 网络和 NAT 网络很相似，不同的地方就是 Host-only 网络没有 NAT 服务，所以虚拟网络不能连接到 Internet。主机和虚拟机之间的通信是通过VMware Network Adepter VMnet1 虚拟网卡来实现的。此时如果想要虚拟机上外网则需要主机联网并且网络共享。
Host-Only模式其实就是NAT模式去除了虚拟NAT设备，然后使VMware Network Adapter VMnet1虚拟网卡连接VMnet1虚拟交换机来与虚拟机通信的，Host-Only模式将虚拟机与外网隔开，使得虚拟机成为一个独立的系统，只与主机相互通讯。

## 新的解决方案

IP会变，我们可以通过设置静态 IP ，这样的话 IP 就不会发生改变，同时设置 NAT 模式，这样就可以通网络了，又可以愉快地用 YUM 在线安装软件，这不比仅主机模式香！！！

### 如何设置

VM 虚拟机右上方  ->  编辑  ->  虚拟网络编辑器  ->  点击更改设置。

![](https://ssuublog.oss-cn-shenzhen.aliyuncs.com/typecho/vm8%E9%9D%99%E6%80%81ip/vm%E8%AE%BE%E7%BD%AE.png)

点击选择 VMnet8   ->   取消勾选 使用本地 DHCP服务。

![](https://ssuublog.oss-cn-shenzhen.aliyuncs.com/typecho/vm8%E9%9D%99%E6%80%81ip/vm8%E8%AE%BE%E7%BD%AE2.png)

点击  NAT设置，记录一下 子网IP ，子网掩码，以及网关。后面设置的时候会用到

![](https://ssuublog.oss-cn-shenzhen.aliyuncs.com/typecho/vm8%E9%9D%99%E6%80%81ip/vm8%E8%AE%BE%E7%BD%AE3.png)

```xml
cd /etc/sysconfig/network-scripts/   #进入到这个目录后ls查看一下。
vim ifcfg-ens32
```

![](https://ssuublog.oss-cn-shenzhen.aliyuncs.com/typecho/vm8%E9%9D%99%E6%80%81ip/static.png)

- 把 BOOTPROTO 改成 static 
- IPADDR 是要设置的静态 IP 地址。  ！！！192.168.23.0这个是从上面可知的，但是我们的设置范围建议是192.168.23.3~192.168.23.253之内的。就是最后那个 0  的范围建议 3~253.
- NETMASK 是 子网掩码   （上面截图可知）
- GATEWAY 是  网关          （上面截图可知）
- DNS1 是 DNS，这里的223.5.5.5是阿里的DNS，这个直接加上去就好了。

![](https://ssuublog.oss-cn-shenzhen.aliyuncs.com/typecho/vm8%E9%9D%99%E6%80%81ip/ens32.png)

```xml
reboot #重启设备
ping baidu.com  #测试行不行
```

Window设置  ->  网络和Internet  ->  以太网  ->  更改适配器选项  ->  选择VM8  ->  属性  ->  Internet版本协议4（tcp/ipv4）

使用下面的 IP地址，子网掩码和网关以及DNS填上面的那个（和虚拟机设置一样），IP不能和虚拟机设置的一样，但是要符合上面说的范围。然后就可以使用 SSH 连接虚拟机了。步骤到此结束。

![](https://ssuublog.oss-cn-shenzhen.aliyuncs.com/typecho/vm8%E9%9D%99%E6%80%81ip/win.png)

