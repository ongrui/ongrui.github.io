# crontab


## 作用

`crontab`在`Linux`系统中用于执行定时任务，比如服务巡检，日志备份清理，数据库备份

在`Centos7`中，最小安装会有该服务，服务名是`crond`



`Crontab`的任务管理在`/etc`目录下的`crontab`文件中

## `crontab`文件详解

```shell
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# | .------------- hour (0 - 23)
# | | .---------- day of month (1 - 31)
# | | | .------- month (1 - 12) OR jan,feb,mar,apr ...
# | | | | .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# | | | | |
# * * * * * user-name command to be executed
```

### 顶部环境相关部分

- `SHELL=/bin/bash`：表示使用`/bin/bash`解释命令
- `PATH=/sbin:/bin:/usr/sbin:/usr/bin`：表示到哪些目录寻找命令执行程序
- `MAILTO`：当有任务执行有输出时，输出内容将发送到那个用户的邮箱，为空则不发送

### 底部定时任务配置部分

- 下面是整个文件的核心部分

```shell
# .---------------- minute (0 - 59)
# | .------------- hour (0 - 23)
# | | .---------- day of month (1 - 31)
# | | | .------- month (1 - 12) OR jan,feb,mar,apr ...
# | | | | .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# | | | | |
# * * * * * user-name command to be executed
```

#### 时间部分

`* * * * *`表示的是时间配置

- 第一个*表示分钟，取值范围在0~59之间的整数
- 第二个*表示小时，取值范围在0~23之间的整数
- 第三个*表示日期，根据下面指定月份中的天数取值
- 第四个*表示月份，取值范围在1~12之间的整数
- 第五个*表示星期，取值范围在1~7之间的整数

特殊符号：

- `*`：代表规则范围内的任意值，如果设置该值的是分钟，则在满足其他条件下的情况下，每分钟都会执行
- `,`：将指定的值隔开，如在分钟中使用，`1,18,23`，在满足其他条件情况下，在1、18、23分钟是都会执行
- `-`：用于指定取值范围，如在分钟中使用，`1-5`，则表示的是1、2、3、4、5分钟
- `/`：用于指定时间的执行频率，如在分钟中使用，`0-30/2`，则表示的是在30分钟之内，每隔两分钟执行一次

#### 时间配置验证

`https://crontab.guru/`

上面网址会对配置的时间验证

#### 命令部分

`user-name command to be executed`表示的是用户和执行的命令

因为环境变量的原因，可能有很多在终端执行的命令，放到`crontab`里面执行不了，所以最好写绝对路径

环境变量的问题下面会详细写下，现在先写一个例子

```
# .---------------- minute (0 - 59)
# | .------------- hour (0 - 23)
# | | .---------- day of month (1 - 31)
# | | | .------- month (1 - 12) OR jan,feb,mar,apr ...
# | | | | .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# | | | | |
# * * * * * user-name command to be executed
  */2 * * * * /usr/sbin/ntpdate s1a.time.edu.cn
```

上面的例子是每两分钟同步一次`s1a.time.edu.cn`的时间


