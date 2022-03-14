# Contos 8 yum 源失效的解决办法

centos 8 官方停止维护了，yum 源自然而然的就会失效。这边文章教你如何重新配置 yum 源。本文使用的是阿里云的源

### 进到目录

```shell
cd /etc/yum.repos.d
```

进到这个目录之后，有很多以 repo 结尾的文件。记住：这些文件都要改动。

### 修改baseurl

```shell
[BaseOS]
name=Qcloud centos OS - $basearch
baseurl=https://mirrors.aliyun.com/centos-vault/8.3.2011/BaseOS/$basearch/os/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
```

我们打开这些 repo 结尾的文件，有 baseurl 这个参数的，我们就把他改了。这里唯一要变的是 83.2011。

```shell
[root@hadoop102 yum.repos.d]# cat /etc/redhat-release
CentOS Linux release 8.3.2011
```

这里我查看了我的发行版本的信息，是 8.3.2011。如果你的是 8.5.2111。

那么你需要把 baseurl 上面的 8.3.2011 替换成 8.5.2111。

### 注释所有 mirrorlist

每一次 repo 结尾的文件都需要修改，见到 mirrorlist 就要把他给注释掉。同时下面原本的 baseurl 注释要去掉，打开 baseurl 这个配置。类似下面这样

```shell
[devel]
name=CentOS Linux $releasever - Devel WARNING! FOR BUILDROOT USE ONLY!
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=Devel&infra=$infra
baseurl=https://mirrors.aliyun.com/centos-vault/8.3.2011/Devel/$basearch/os/
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
```

### 关于 gpgkey

```shell
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
```

出现 gpgkey 的参数，全部改成。

### 最后

```shell
yum clean all
yum makecache
```

等待即可。

### 总结

- 全部 repo 文件都得改
- 改的地方有上面三处









