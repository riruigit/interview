## 开源的仓库地址

相关软件出处下载地址：

https://github.com/Fndroid/clash_for_windows_pkg

https://github.com/Kr328/ClashForAndroid

https://github.com/BoyceLig/Clash_Chinese_Patch

相关视频教程：（需要科技）

https://www.youtube.com/watch?v=V1oSmz1k29Y&ab_channel=%E7%A7%91%E6%8A%80%E5%88%86%E4%BA%AB

相关的配置文档：

https://docs.cfw.lbyczf.com/contents/tun.html#windows

## 更快捷的工具

在没写这个文章之前，我是一直用的是 v2 的。最近付费购买了张机票，机场方面貌似不支持了v2。没得办法开始体验了猫咪，没想到这个软件用了之后就回不去了。

第一个仓库地址是猫咪的win下的客户端。虽然不是官方的仓库，但是根据这个标星数来看，还是可以的。

第二个是手机下的客户端。

第三个是汉化的仓库，关于怎么把软件汉化，可以看项目的readme。

## tun模式

猫咪提供了两种代理模式，一种是普通代理，一种是模拟一个网卡实现全局的代理。

我之前的需求是：对于国内的网站不走代理，然后对于国外的走代理加速，虽然可以使用v2设置绕过大陆的模式，但是在部分网站打开仍然可以感觉到了卡顿，但是用了猫咪之后，感觉很是丝滑。毕竟现在对这个已经重度依赖了。

如何设置：服务模式中安装好网卡驱动（地球会变绿色的），这时候会在win设置里面的网络适配器中虚拟出一个网卡的。然后就是开启tun模式。开了这个模式就需要把系统代理这个模式关掉。

用下面的方式修改配置文件，去掉原有的dns配置。

```yaml
dns:
  enable: true
  enhanced-mode: fake-ip
  nameserver:
    - 8.8.8.8 # 真实请求DNS，可多设置几个
    - 114.114.114.114
# interface-name: WLAN # 出口网卡名称，或者使用下方的自动检测
tun:
  enable: true
  stack: gvisor # 使用 system 需要 Clash Premium 2021.05.08 及更高版本
  dns-hijack:
    - 198.18.0.2:53 # 请勿更改
  auto-route: true
  auto-detect-interface: true # 自动检测出口网卡
```

## 关于配置文件的修改

在最新的版本里面，直接编辑配置文件，但是如果你刷新了，这个配置文件就会被重置了。修改配置文件，应该要打开合并，初始化一个配置文件，然后进去编辑修改合并模式。修改右边边框的文件。

然后后面刷新的时候，就会自动合并上你修改的内容了。

