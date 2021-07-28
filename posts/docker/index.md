# docker


## 什么是Docker

docker是一款轻量级的虚拟化工具，它会提供一个虚拟环境，名为容器。

容器需要运行镜像来生成。

构建镜像可以使用docker commit命令和编辑Dockerfile文件后使用docker build命令构建

在linux中，通过命名空间与物理机的进程、网络、文件系统、CPU内存等资源的隔离。

容器可以打包成镜像后移动到另一个物理机器中，然后运行该镜像即可生成一个容器。

这样就完成了容器的移植。

***--- 下面内容摘自docker官网***

将软件打包到标准化单元进行开发、装运和部署

容器是软件的标准单元，可将代码及其所有依赖项打包，以便应用程序从一个计算环境快速可靠地运行到另一个计算环境。Docker 容器镜像是一个轻量级、独立、可执行的软件包，包括运行应用程序所需的一切：代码、运行时间、系统工具、系统库和设置。

容器镜像在运行时变成容器，在docker容器的情况下 - 图像成为容器时，他们运行在docker引擎。针对 Linux 和 Windows 的应用程序，无论基础设施如何，容器化软件始终运行相同。容器将软件与环境隔离开来，并确保它工作一致，尽管开发和分期之间存在差异。

在码头发动机上运行的码头集装箱：

- **标准：**Docker 为容器创建了行业标准，因此它们可以在任何地方移植
- **重量轻：**容器共享机器的操作系统内核，因此不需要每个应用程序的操作系统，从而提高服务器效率并降低服务器和许可成本
- **安全：**在容器中应用更安全，Docker 提供业内最强的默认隔离功能

## 常用命令

- 列出所有镜像

```
docker images
```

- 运行镜像

```
docker run
	-it ... /bin/bash以交互模式运行
	--name 运行镜像后生成的容器名称
	-p 8080:8081 将容器中的8081端口映射到本机的8080端口
	-v /home/data:/data 将容器中的/data目录映射到本地的/home/data目录
```

- 列出所有容器

```
docker ps -a
```

- 列出正在运行的容器

```
docker ps
```

- 启动容器

```
docker start 容器id/容器名称
```

- 停止容器

```
docker stop 容器id/容器名称
```

- 重启容器

```
docker restart 容器id/容器名称
```

- 进入容器

```
docker exec -it 容器id/容器名称 bash
```

- 容器和物理机相互拷贝文件

  - 将容器内/usr/local/redis/conf/redis.conf拷贝到物理机的当前目录\

    ```
    docker cp 容器id/名称:/usr/local/redis/conf/redis.conf ./
    ```

  - 将物理机当前目录下的redis.conf文件拷贝到容器内的/etc目录下

    ```
    docker cp ./redis.conf 容器id/名称:/etc/
    ```

- 删除容器（只能删除停止的容器）

```
docker rm 容器id/容器名称
```

- 删除镜像

```
docker rmi imageId
```

## 导出导入

### 导出

1.提交容器修改，会新创建一个镜像

```bash
docker commit 10874ee33238 oracle:6120bak
```

2.查看保存的镜像

```bash
docker images
```

3.将镜像另存一个备份文件

```bash
docker save -o oracle.tar oracle:6120bak
```

### 导入

4.导入镜像

```bash	
docker load < oracle.tar
```

```bash
docker load -i oracle.tar
```

## 构建镜像

构建镜像有两种方式，一种是基于已有镜像生成的容器，使用commit命令提交对容器的修改

一种是使用Dockerfile编译生成镜像

### commit

commit命令是依照当前容器创建一个镜像

可以理解为当对初始镜像run之后生成的容器做了修改之后，就可以使用commit命令来提交对该容器的修改

比如说一个centos7镜像，在run之后生成的容器上安装了一个gcc，此时就可以使用commit命令来提交对该容器的修改，此时commit的只是初始状态可当前状态之间有差异的部分，所以commit操作就很轻量

#### 示例

- 运行一个镜像

```
[root@pacs ~]# docker run -dit --name centos7 ansible/centos7-ansible:1.8
fb26e1fd669bbccdd2fb0fdd1e1a312dcc815df4c45969de662416c71d752f39
```

- 查看正在运行中的容器

```
[root@pacs ~]# docker ps
CONTAINER ID  IMAGE                        COMMAND     CREATED          STATUS          PORTS     NAMES
fb26e1fd669b  ansible/centos7-ansible:1.8 "/bin/bash"  37 seconds ago   Up 33 seconds             centos7
```

- 进入centos7容器

```
[root@pacs ~]# docker exec -it centos7 bash
[root@fb26e1fd669b ansible]# 
```

- 退出centos7容器

```
[root@fb26e1fd669b home]# exit
exit
```

- 安装gcc

```
步骤略
```

- 查看gcc是否安装成功

```
gcc version 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC) 
```

- 提交对容器的修改

```
[root@pacs gcc]# docker commit -m "intall gcc" fb26e1fd669b centos7:v2
sha256:48439ec1ff536277c2a4870ff068c2d0415751f9a21ee958af0bf0d873e0f9aa

参数说明：
-m：指定提交说明信息
fb26e1fd669b：容器id
centos7:v2：指定image名称和版本号
```

- 查看生成的镜像

```
[root@pacs gcc]# docker images
REPOSITORY                  TAG             IMAGE ID       CREATED          SIZE
centos7                     v2              48439ec1ff53   12 seconds ago   705MB
```

- 将旧的容器停止并删掉

```
[root@pacs gcc]# docker stop fb26e1fd669b
fb26e1fd669b
[root@pacs gcc]# docker rm fb26e1fd669b
fb26e1fd669b
```

- 运行新的镜像

```
[root@pacs gcc]# docker run -dit --name centos7 centos7:v2
c1f9774ca85e8fa652e29adaa39e62e2909e6dfb604aac09fb1a8f11e2a77d1d
```

- 查看生成的容器

```
[root@pacs gcc]# docker ps
CONTAINER ID   IMAGE          COMMAND        CREATED              STATUS                PORTS   NAMES
c1f9774ca85e   centos7:v2     "/bin/bash"    About a minute ago   Up About a minute             centos7
```

- 进入容器

```
[root@pacs gcc]# docker exec -it centos7 bash
[root@c1f9774ca85e ansible]# 
```

- 查看gcc是否安装

```
[root@c1f9774ca85e ansible]# gcc -v 
Using built-in specs.
...
gcc version 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC)
```

### Dockerfile

dockerfile是一个文件，该文件中保存了将要创建的容器信息

然后使用docker build创建镜像

#### Dockefile:

dockerfile可分为下面几个部分：

- 基础镜像信息
- 维护者信息
- 镜像操作指令
- 容器启动时执行指令

#### Dockerfile中的常用指令

- FROM：指定基础镜像，如创建一个nginx镜像，基础镜像可以指定centos或ubuntu，表示在该镜像的基础上构建，官方指定的是debian
- ENV：用来定义镜像的环境变量
- LABEL：指定镜像标签，一个镜像可以有多个标签
- RUN：运行该镜像时执行的命令
- ADD：添加文件到镜像中，如果是tar文件，会自动提取到指定的目录
- WORKDIR：为RUN、CMD、COPY和AND设置当前工作目录
- VOLUME：指定挂载到宿主机那个目录。一般用在docker run中，在运行镜像时使用-v指定目录挂载
- USER：指定运行容器的用户
- EXPOSE：指定容器运行后需要监听的端口
- CMD：指定启动容器时运行的命令或脚本，只能有一条被执行，有多条时则执行最后一条

#### 示例：

```dockerfile
FROM centos:7

LABEL maintainer="<rui.u@foxmail.com>"

RUN yum -y install gcc \
    yum -y install make

COPY redis-5.0.10.tar.gz /usr/local

RUN tar -zxvf /usr/local/redis-5.0.10.tar.gz -C /usr/local && rm -f /usr/local/redis-5.0.10.tar.gz && mv /usr/local/redis-5.0.10 /usr/local/redis

WORKDIR /usr/local/redis

RUN make && make install && rm -rf redis.conf

COPY redis.conf ./

EXPOSE 6379

WORKDIR /

ENTRYPOINT ["/usr/local/redis/src/redis-server","/usr/local/redis/redis.conf"]
CMD []
```

- 把Dockerfile文件中所需的文件放到与Dockerfile的同级目录，

```
[root@pacs dockerfi]# ls
Dockerfile  redis-5.0.10.tar.gz  redis.conf
```

- 执行命令编译镜像文件

  ```
  [root@pacs dockerfi]# docker build -t redis:rui ./
  ```

  - -t：指定编辑好的镜像的name和tag
  - ./：指定Dockerfile的位置

- 编译完成之后使用docker images查看编译好的镜像

```
[root@pacs dockerfi]# docker images
REPOSITORY                   TAG             IMAGE ID       CREATED         SIZE
redis                        rui             c63f66127130   9 minutes ago   556MB
```

- 执行下面命令运行镜像

```
[root@pacs dockerfi]# docker run -d --name redis -p 6379:6379 redis:rui
474523dd27912b981a828789c27d640891dbf350e96b00f575e5479da1bdf636
[root@pacs dockerfi]# 
```

- 查看正在运行的容器

```
[root@pacs dockerfi]# docker ps
CONTAINER ID   IMAGE       COMMAND                  CREATED         STATUS         PORTS                       NAMES
474523dd2791   redis:rui   "/usr/local/redis/sr…"   5 seconds ago   Up 2 seconds   0.0.0.0:6379->6379/tcp      redis
```

- 进入容器

```
[root@pacs dockerfi]# docker exec -it redis bash
[root@474523dd2791 /]#
```

- 测试redis

```
[root@474523dd2791 /]# redis-cli 
127.0.0.1:6379> ping
(error) NOAUTH Authentication required.
127.0.0.1:6379> quit
[root@474523dd2791 /]# redis-cli 
127.0.0.1:6379> auth admin
OK
127.0.0.1:6379> ping
PONG
127.0.0.1:6379> set test 1
OK
127.0.0.1:6379> get test
"1"
127.0.0.1:6379> del test
(integer) 1
127.0.0.1:6379> get test
(nil)
127.0.0.1:6379> 
```

#### 遇到的麻烦

- docker run 后面不能跟/bin/bash，否则会覆盖掉Dockerfile中cmd后面执行的命令，

  也会导致容器进程不能维持运行，

  可是我运行的参数写到了ENTRYPOINT后面

  pull的其它镜像一般都是使用-dit /bin/bash方式运行，并没有发现有什么问题

  可能还是我的Dockerfile写的有问题

- 还是需要继续学习啊

## 学习资料

docker官方文档：

https://docs.docker.com/engine/reference/builder/#shell

csdn：

https://blog.csdn.net/wangziyang777/article/details/114277452

https://blog.csdn.net/guyan0319/article/details/81020252

https://blog.csdn.net/persistencegoing/article/details/93713869

redis官方镜像dockerfile（看的迷迷糊糊:sob:）：

https://github.com/docker-library/redis/blob/147762b57f4d4391ba6cf8fbd1e7590a606643ef/5/Dockerfile

dockerhub：

https://hub.docker.com/


