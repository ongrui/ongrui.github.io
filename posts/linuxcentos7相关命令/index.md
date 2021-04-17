# linux(Centos7)相关命令 持续更新...


# 开篇啰嗦

啥时候更新时间间隔超过三个月我就把持续更新去掉

话说作为一个测试，第一篇文章是不是应该得是测试相关的

唉，在公司学的最多的就是linux

先写这个吧，明天再写一篇测试相关的

下面都是我目前能想到或工作中常用的

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

## 日志查看

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
  - `-n`：打印制定行
  - `'10,20p'`：指定的行号
- 方式二：使用：`cat -n 日志文件名称 | head -n 20 | tail -n +10`
  - `cat -n`：打印日志显示行号
  - `head -n 20`：打印到20行
  - `tail -n +10`：从第10行开始打印


