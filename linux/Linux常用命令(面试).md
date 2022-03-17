# Linux 常用命令

## 常见的指令

cat vi vim ls mkdir touch cp mv tar find tail

### 查看日志

日志文件一般很长，这个时候可以选择从后面开始加载。

tail -n 20 xxx.log	tail -f xxx.log   前者是显示后面的20行，后者是显示后10行（默认），然后写进来的就实时读取。

### 打包与解压

注意一点，linux上的打包和压缩是两个操作，这两者需要区别开来，不过一般情况下使用的都是 tar 这个指令。

解压命令 ： tar -zxvf xxxx.tar.gz  -C /xxx/xxx

其中 z 表示通过--gzip或--ungzip 处理文件。x表示释放打包文件。v表示显示执行的过程。f 表示指定文件。

压缩指令 ： tar -czvf  xxx.tar.gz  xxx

和上面几乎一样，打包用 c ,解压用 x

### 查看磁盘和文件大小

查看磁盘的 df -h

查看文件的 du -hls

### 杀死进程

一般情况下要先找到进程的进程号，然后再kill 掉。

ps -ef | grep xxx  kill -9 xxxx

### 查看所有组和用户

Linux 上都是文件。组文件对应  /etc/group   用户对应 /etc/passwd

### 修改文件权限和归属用户

这两者的命令并不相同，chmod 是修改文件的权限，执行 ll 指令的时候可以看出，前面那部分（-rw-r--r--）就是权限，后面是文件归属的用户以及组。

如果是修改执行权限这些，chmod +777 xxxx

这里用数字就好了，4 2 1 对应下来读写执行。

如果修改文件的归属组和用户，就需要 chown ,其中 -r 是递归执行。

chown -R root:root  /tmp/sco

### 查看网络端口占用情况

netstat -ntlp | grep 2240

grep 可以跟进程名字，也可以加端口号。

### 查看进程

ps -ef | grep xxx

ps -aux

