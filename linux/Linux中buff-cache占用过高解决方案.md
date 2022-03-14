# Linux中buff-cache占用过高解决方案

我们在使用`free -h`查看系统内存的时候,有时间会发现`buff/cache`很高

### 至于为什么？自行百度吧

```shell
sync
echo 1 > /proc/sys/vm/drop_caches;
echo 2 > /proc/sys/vm/drop_caches;
echo 3 > /proc/sys/vm/drop_caches;
```

