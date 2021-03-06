# linux(Centos7)相关命令 持续更新...


## 文件操作相关

### 文件拷贝 cp

- 拷贝文件：```cp 要拷贝的文件 目标路径```
- 拷贝文件`redis.conf`到`/etc/`目录下

```tex
[root@rui redis-6.2.0]# cp redis.conf /etc/
[root@rui redis-6.2.0]# ls /etc/redis.conf 
/etc/redis.conf
```

- 拷贝文件夹：`cp -r 要拷贝的文件夹 目标路径`
- 拷贝文件夹`redis-6.2.0`到`/usr/local/`目录下

```tex
[root@rui home]# cp -r redis-6.2.0/ /usr/local/
[root@rui home]# ls /usr/local/
bin  games    lib    libexec  redis-6.2.0  share
etc  include  lib64  proc     sbin         src
```

### 文件移动 mv

- 移动文件夹/文件：`mv 要移动的文件夹或者是文件 目标路径`
- 移动文件`redis.conf`到`src`目录下

```tex
[root@rui redis-6.2.0]# mv redis.conf src/
[root@rui redis-6.2.0]# ls src/redis.conf 
src/redis.conf
```

### 文件查找

- 方式一：`find 要查找的路径 -name 要查找的文件`
- `find`会遍历指定路径下的所有文件，然后返回符合条件的，支持模糊查询

```tex
[root@rui redis-6.2.0]# find ./ -name redis.c*
./src/redis.conf
```

- 方式二：`locate 要查找的文件`
- `locate` 在`/var/lib/mlocate/`目录下存放着一个数据库文件，其实使用`locate`查询是查询的数据库，所以`locate`非常快
- `locate`会返回所有包含文件名的文件

```tex
[root@rui redis-6.2.0]# locate /home/redis-6.2.0/src/red
/home/redis-6.2.0/src/redis-benchmark
/home/redis-6.2.0/src/redis-benchmark.c
/home/redis-6.2.0/src/redis-benchmark.d
/home/redis-6.2.0/src/redis-benchmark.o
/home/redis-6.2.0/src/redis-check-aof
/home/redis-6.2.0/src/redis-check-aof.c
/home/redis-6.2.0/src/redis-check-aof.d
/home/redis-6.2.0/src/redis-check-aof.o
/home/redis-6.2.0/src/redis-check-rdb
/home/redis-6.2.0/src/redis-check-rdb.c
/home/redis-6.2.0/src/redis-check-rdb.d
/home/redis-6.2.0/src/redis-check-rdb.o
/home/redis-6.2.0/src/redis-cli
/home/redis-6.2.0/src/redis-cli.c
/home/redis-6.2.0/src/redis-cli.d
/home/redis-6.2.0/src/redis-cli.o
/home/redis-6.2.0/src/redis-sentinel
/home/redis-6.2.0/src/redis-server
/home/redis-6.2.0/src/redis-trib.rb
/home/redis-6.2.0/src/redis.conf
/home/redis-6.2.0/src/redisassert.h
/home/redis-6.2.0/src/redismodule.h
```

然后新建一个文件，再查询一次

```tex
[root@rui redis-6.2.0]# > /home/redis-6.2.0/src/testfile
[root@rui redis-6.2.0]# locate /home/redis-6.2.0/src/testf
[root@rui redis-6.2.0]# ll /home/redis-6.2.0/src/testfile 
-rw-r--r--. 1 root root 0 Apr 10 22:37 /home/redis-6.2.0/src/testfile
```

可以看到，没有查到，因为`locate`的数据库不是实时更新的，可以使用`updatedb`手动更新，也可以把更新命令配置到系统的任务计划表，定时执行

```tex
[root@rui redis-6.2.0]# updatedb
[root@rui redis-6.2.0]# locate /home/redis-6.2.0/src/testf
/home/redis-6.2.0/src/testfile
```

可以看到更新之后就查到了

2021-05-13：

### 文件权限相关

```
[root@rui nginx]# ll
total 12
drwx------ 2 root root    6 Mar  8 11:49 client_body_temp
drwxr-xr-x 2 root root 4096 May 10 11:39 conf
drwx------ 2 root root    6 Mar  8 11:49 fastcgi_temp
drwxr-xr-x 2 root root   38 Mar  8 10:50 html
drwxr-xr-x 2 root root   55 Mar 29 10:01 logs
-rw-r--r-- 1 root root 4796 May 11 20:14 nginx.conf
drwx------ 3 root root   14 Apr 29 10:04 proxy_temp
drwxr-xr-x 2 root root   18 Mar  8 10:50 sbin
drwx------ 2 root root    6 Mar  8 11:49 scgi_temp
drwx------ 2 root root    6 Mar  8 11:49 uwsgi_temp
```

- total 12

  linux的数据存储是以块（block）为单位的，块可以理解成一个容器，每个块的大小可以用`getconf PAGESIZE`命令查看

  每个块的容量是4096B，也就是4K

```go
drwx------ 2 root root    6 Mar  8 11:49 client_body_temp
```

`drwx------`

- d：表示文件夹，如果是-，则代表是文件

- r：表示有可读权限，用数字表示为4

- w：表示有可写权限，用数字表示为2

- x：表示有可执行权限，用数字表示为1

- rwx：文件权限

  - 一个rwx可以理解为为一组权限，一个文件有三组，第一组为文件所属用户，第二组为所属用户组内的其他用户，第三组为其他用户
  - 如果显示为"-"符号则表示没有权限

  - 设置权限可以使用`chmod` 

  `chmod 741 filename`

  - 741：
    - 文件所属用户有该文件的可读、可写、可执行权限
    - 文件所属用户组内的其它用户有该文件的可读权限
    - 其他用户有该文件的可执行权限

`drwx------ 2 root root 6 Mar  8 11:49 client_body_temp`

- 2：硬连接个数
- 第一个root：文件所属用户
- 第二个root：文件所属用户的组
- 6：文件大小
- Mar  8 11:49：最后修改时间
- client_body_temp：文件名/目录名

## `vim`相关

### 进入编辑模式

- 按`i`进入编辑模式后光标会在当前位置
- 按`I`进入编辑模式后光标会在当前行行首
- 按`a`进入编辑模式后，光标在当前字符后
- 按`A`进入编辑模式后，光标会在当前行行尾
- 按`o`进入编辑模式后，会在当前行下一行新开一行
- 按`O`进入编辑模式后，会在当前行上一行新开一行

`我用的最多的是i`

### 退出编辑模式

- 按`ESC`退出编辑模式

### 查找

- 使用：`/要查找的内容`
- 使用`vim /文件名`打开文件后，输入`/`要查找的内容即可，比如下面我要在`redis.conf`配置文件中查找`bind`关键字

{{< image src="/img/image-20210410145934938.png">}}

- 然后可以输入回车，会查找到该文件中第一个`bind`，可以输入`n`定位到下一个，输入`N`定位到上一个

- 取消高亮可以输入`nohlsearch`，简写为`noh`，也可以输入`set nohlsearch` ，简写为`set noh`

### 替换

- 把文件中的所有`127.0.0.1`替换为`120.5.5.153`
  - 使用：`:%s/127.0.0.1/120.5.5.153/g`

- 把当前行的`127.0.0.1`替换为`120.5.5.153`
  - 使用：`:S/127.0.0.1/120.5.5.153/g`

`还有很多替换相关的命令，这里只是写了常用的`

### 退出

- 退出不保存`:q`
- 强制退出不保存`:q!`
- 保存并退出`:wq`
- 强制保存并退出`:wq!`

## 查看日志

### 实时刷新查看日志

- 方式一：使用：`tail -f -n 10 日志文件名称`
  - `-f`：实时刷新
  - `-n`：行数关键字，后面跟行数，表示从后10行开始查看
- 方式二：使用：`tailf 日志文件名称`
  - 默认从后十行开始查看

### 查看日志前10行

- 使用：`head -n 行数 日志文件名称`
  - `-n` ：行数关键字

### 查看第10行到第20行

- 方式一：使用：`sed -n '10,20p' 日志文件名称`
  - `-n`：打印指定行
  - `'10,20p'`：指定的行号
- 方式二：使用：`cat -n 日志文件名称 | head -n 20 | tail -n +10`
  - `cat -n`：打印日志显示行号
  - `head -n 20`：打印到20行
  - `tail -n +10`：从第10行开始打印

2021.04.20

## 查看进程

### ps命令

ps （英文全拼：process status）命令用于显示当前进程的状态 --摘自菜鸟教程

#### 参数

- -a：列出所有进程
- -w：显示加宽，可以显示更多内容
- -u：以用户为主的进程状态
- -x：显示当前用户在所有终端下的进程信息，通常和参数`a`一起使用
- -e：显示系统内所有的进程信息
- -l：使用长格式显示进程信息
- -f：使用完整的格式显示进程信息

#### 筛选

- 在输入了上面命令后添加`|grep processName/processID`
- `ps -aux |grep mysql`

### 根据进程查找文件

- 当因为某种原因找不到服务的存放位置时，可以使用上面的命令先查看进程的ID

- 然后使用命令：`pwdx processID`命令查找该进程文件存放的位置

  ```shell
  [root@rabbitmq1 redis]# ps -aux |grep redis
  redis     4356  0.1  2.3 323184 133272 ?       Ssl  4月19  25:12 /usr/bin/redis-server *:6379
  root      7194  0.0  0.0 112736   996 pts/1    S+   11:23   0:00 grep --color=auto redis
  [root@rabbitmq1 redis]# pwdx 4356
  4356: /var/lib/redis
  ```

## 查看端口号是否被监听

```shell
[root@pacs local]# ss -tunlp |grep 3306
tcp    LISTEN     0      128       *:3306      *:*    users:(("docker-proxy",pid=28320,fd=4))
```

2021.07.10

## 查看文件系统硬盘使用情况

- 使用df来查看硬盘使用情况

### 参数

- a：显示所有文件信息
- m：以MB为单位显示容量
- k：以KB为单位显示容量
- h：以KB、MB、GB为单位自行显示容量
- T：显示该分区的文件系统名称
- i：不用硬盘容量显示，而是以含有inode的数量来显示

使用的最多的是df -h

```
[root@pacs ~]# df -h
Filesystem               Size  Used Avail Use% Mounted on
devtmpfs                 7.8G     0  7.8G   0% /dev
tmpfs                    7.8G     0  7.8G   0% /dev/shm
tmpfs                    7.8G  777M  7.0G  10% /run
tmpfs                    7.8G     0  7.8G   0% /sys/fs/cgroup
/dev/mapper/centos-root   50G   15G   36G  29% /
/dev/mapper/centos-home  167G   83G   84G  50% /home
/dev/xvda1               497M  219M  279M  45% /boot
tmpfs                    1.6G   12K  1.6G   1% /run/user/42
tmpfs                    1.6G     0  1.6G   0% /run/user/385
tmpfs                    1.6G     0  1.6G   0% /run/user/0
/dev/dm-5                 30G  550M   30G   2% /home/apps/docker/devicemapper/mnt/716b06271724538bfaad350c0a054dde7e54db34e28c417f07842564d8a14021
/dev/dm-6                 30G  526M   30G   2% /home/apps/docker/devicemapper/mnt/cd888eb5fe250ab255fc13646a5fd4de943b2b1bf162f64a7146697378ae9424
/dev/dm-7                 30G   12G   19G  38% /home/apps/docker/devicemapper/mnt/4e95d08a9f25293974b18da280484a55cd1359e593d60c1a0db5697642c24c13
/dev/dm-8                 30G  493M   30G   2% /home/apps/docker/devicemapper/mnt/652cb0933f40cea24bfd8df3ec43e63f02e65a51cd48bbd4fd4d8a617da2f7ac
/dev/dm-9                 30G  494M   30G   2% /home/apps/docker/devicemapper/mnt/3c74dba554f0e812152b80e1fc6d12865530ed573b57059d3cb01e0f3099c704
/dev/dm-10                30G  9.0G   22G  30% /home/apps/docker/devicemapper/mnt/8fc80177eea5c6f9565c102a9c1d234796e2ee9d9f444067522ed7d36765d0b4
```

20210728

## 用户组和用户操作

### 创建和删除

```bash
# 添加用户组
groupadd tester

# 在tester用户组下创建一个用户
useradd -g tester testA

# 给testA创建密码
passwd testA

# 修改用户密码
# 修改root用户:
passwd
# 修改其它用户（需要使用root用户修改）:
passwd testA

# 查看用户属于那个组
groups testA
id testA

# 删除用户
userdel testA
# 删除用户组
groupdel tester
```

### 权限

- 将/data/test/start.sh文件的拥有者赋予用户testA

```shell
chown testA:tester /data/test/start.sh
```

- 将/data/test目录及其下面的所有文件的拥有者赋予用户test1（使用-R参数）

```shell
chown -R testA:tester /data/test
```




