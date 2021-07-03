# 工作记录之rabbitmq镜像集群部署(短篇)


记录一次工作中搭建rabbitmq集群遇到的问题

## rabbitmq是啥

erlang语言编写的开源的用来做消息队列的系统

## 集群部署流程（centos7）

erlang部署略

rabbitmq单节点部署略

- 下载与自己服务器相应的包，我下载了一个Unix的包，然后日志提示读取磁盘信息错误，centos7下载el7的包即可

192.168.1.10

192.168.1.11

192.168.1.12

配置集群

10为主节点

- 1.在/etc/hosts文件中配置三台机器的主机名和IP映射，三台服务器都要配置

```
192.168.1.10 mqa

192.168.1.11 mqb

192.168.1.12 mqc
```

- 2.修改/var/lib/rabbitmq/.erlang.cookie文件中的内容，需要三台服务器一致

  该文件其他人的权限必须是0

- 3.在mqb和mqc执行下面命令添加到mqa节点

```
systemctl start rabbitmq-server # 启动服务
rabbitmqctl stop_app            # 停止rabbitmq应用
rabbitmqctl reset               # 重置节点，该操作会删除该节点的集群信息，持久化信息，用户信息
rabbitmqctl join_cluster rabbit@mqa # 将该节点添加到mqa节点下，mqa节点必须是开启状态
rabbitmqctl start_app           # 启动rabbitmq应用
rabbitmqctl-plugins enable rabbitmq_management # 开启web管理插件，如果想在浏览器查看集群状态时，每个节点都要启用该配置
```

- 4.在主节点开启web管理插件

```
rabbitmq-plugins enable rabbitmq_management
```

- 5.在浏览器访问主节点IP+15672端口就能看到集群相关信息了

写的很简单，过程很心酸:sob:
