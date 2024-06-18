# 欢迎来到SQL小白的学习笔记（docker基础知识）

*本次学习课程为B站黑马程序员系列课程之“Docker快速入门到项目部署，MySQL部署+Nginx部署+docker自定义镜像+DockerCompose项目实战一套搞定”，这部分有些环节只记录了一些关键的步骤，有很多口语化的表达。
现在将学习笔记记录在此，仅供自己复习，笔记中如有错误，欢迎指出。

[基础部分](https://github.com/shengli666666/MySQL-/tree/main#%E5%9F%BA%E7%A1%80%E9%83%A8%E5%88%86beginning)
	* [1.SQL概述](https://github.com/shengli666666/MySQL-/tree/main#1sql%E6%A6%82%E8%BF%B0)
	* [2.SQL基础以及分类](https://github.com/shengli666666/MySQL-/tree/main#2sql%E5%9F%BA%E7%A1%80%E5%8F%8A%E5%88%86%E7%B1%BB)
	* [3.SQL基础---函数](https://github.com/shengli666666/MySQL-/tree/main#3sql%E5%9F%BA%E7%A1%80---%E5%87%BD%E6%95%B0)
	* [4.基础约束](https://github.com/shengli666666/MySQL-/tree/main#4%E5%9F%BA%E7%A1%80%E7%BA%A6%E6%9D%9F)
	* [5.多表查询](https://github.com/shengli666666/MySQL-/tree/main#5%E5%A4%9A%E8%A1%A8%E6%9F%A5%E8%AF%A2)
	* [6.事务](https://github.com/shengli666666/MySQL-/tree/main#6%E4%BA%8B%E5%8A%A1)

# 第一部分docker常见命令
```shell
（自己整理了一部分基本操作命令）
1.查看docker版本：sudo docker version 或者 docker -v
2.启动docker：systemctl start docker；docker run是重新建立并且启动，与start不一样
3.重启docker：systemctl restart docker；
4.查看docker是否启动：docker images
5.执行docker ps 命令不报错，说明安装成功，或者docker images验证docker好不好用
6.docker ps 默认查看当前正在运行的容器
7.docker ps -a 查看所有容器
8.进入容器内部并修改配置文件： docker exec -it 容器名 bash

docker与镜像
1.查看镜像：docker images
2.下拉镜像： docker pull 镜像名
  sudo docker pull 镜像名:Tag；不加版本号就是拉取最新版
3.删除镜像：docker rmi  镜像名/镜像ID
强制删除： docker rmi -f 镜像名/镜像ID
4.通过加载包来加载镜像：docker load -i XX.tar(镜像保存的位置)
5.保存镜像：docker save 镜像名/镜像ID -o 镜像保存在哪个位置与名字
6.搜索镜像：docker search 镜像名

docker与容器
1.docker ps -a 查看所有容器（包括没运行的）
2.创建容器：
docker run -it -d --name 要取的别名 -p 宿主机端口:容器端口 -v 宿主机文件存储位置:容器内文件位置 镜像名:Tag /bin/bash
参数说明：
-it 表示 与容器进行交互式启动
-d 表示可后台运行容器 （守护式运行）  
--name 给要运行的容器 起的名字  
/bin/bash  交互路径
-p 将容器的端口映射到宿主机上，通过宿主机访问内部端口
-v 将容器内的指定文件夹挂载到宿主机对应位置
3.停止容器；docker stop 容器名/容器ID
4.删除容器：
删除没在运行的容器：docker rm 容器名
删除正在运行的容器：
#删除一个容器
docker rm -f 容器名/容器ID
#删除多个容器 空格隔开要删除的容器名或容器ID
docker rm -f 容器名/容器ID 容器名/容器ID 容器名/容器ID
#删除全部容器
docker rm -f $(docker ps -aq)
```
# 第二部分数据卷
```shell
数据卷是一个虚拟的目录，是容器内目录与宿主机目录之间映射的桥梁，方便操作容器内文件或者方便迁移容器产生的数据。
--- 进入nginx容器 docker exec -it nginx bash
```
## docker案例一
```shell
案例1-利用Nginx容器部署静态资源 需求如下：
创建Nginx容器，修改nginx容器内的html目录下的index.html文件，查看变化·将静态资源部署到nginx的html目录

----分析
zsl@zsl-virtual-machine:~$ docker start nginx
nginx
 docker ps
 输出结果：
CONTAINER ID   IMAGE     COMMAND                   CREATED       STATUS              PORTS                               NAMES
67818f418da0   nginx     "/docker-entrypoint.…"   4 weeks ago   Up About a minute   0.0.0.0:80->80/tcp, :::80->80/tcp   nginx
 docker exec -it nginx bash
root@67818f418da0:/# cd /usr/share/nginx/html
root@67818f418da0:/usr/share/nginx/html# ls
50x.html  index.html
root@67818f418da0:/usr/share/nginx/html# vi index.html
bash: vi: command not found
root@67818f418da0:/usr/share/nginx/html# 

bash: vi: command not found
报错原因是在容器中修改很困难，使用数据卷解决，数据卷是一个虚拟目录，是容器内目录与宿主机目录之间的映射桥梁，但是数据卷保存下的文件是真实的
---- Linux统一保存的目录/var/lib/docker/volumes/下，此时docker会将宿主机和容器中的文件进行双向的绑定，此时直接在宿主机中操作即可，容器内会跟着更新
---- 挂载时文件不存在也可自动创建
--- 挂载时一定是在run时候，已经创建了容器是不可以挂载的

nginx静态资源地址/usr/share/nginx/html

---答案
zsl@zsl-virtual-machine:~$ docker rm -f nginx###先删除
nginx
zsl@zsl-virtual-machine:~$ docker run -d --name nginx -p 80:80 -v html:/usr/share/nginx/html nginx##创建
39c64b853bdc0dd9d96bd20411a7b1c39eab8fc4bc60f99c7642628eb3e043ce
zsl@zsl-virtual-machine:~$ docker ps
CONTAINER ID   IMAGE     COMMAND                   CREATED          STATUS          PORTS                               NAMES
39c64b853bdc   nginx     "/docker-entrypoint.…"   13 seconds ago   Up 12 seconds   0.0.0.0:80->80/tcp, :::80->80/tcp   nginx
zsl@zsl-virtual-machine:~$ docker volumes ls##查看卷
docker: 'volumes' is not a docker command.
See 'docker --help'
zsl@zsl-virtual-machine:~$ docker volume ls
DRIVER    VOLUME NAME
local     e2d80722ec35530d49feb86ec50d7f347b0aaec20bf299c709f7f6eee1152529
local     e4297575bc4ea067adc2674e0b8305a82919db27a94ab2f03e1a12e3d69f3b99
local     html
zsl@zsl-virtual-machine:~$ docker volume inspect html##查看卷创建信息以及宿主机保存位置/var/lib/docker/volumes/html/_data
[
    {
        "CreatedAt": "2024-05-23T11:36:25+08:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/html/_data",
        "Name": "html",
        "Options": null,
        "Scope": "local"
    }
]
zsl@zsl-virtual-machine:~$ cd /var/lib/docker/volumes/html/_data##可以进入宿主机保存的文件位置
zsl@zsl-virtual-machine:~$ ll
zsl@zsl-virtual-machine:~$ cat index.html 即可看到nginx文档
接下来对nginx.html进行修改，可以用vi命令，也可以直接找到html文件双击修改即可
```

## docker案例二
```shell
案例2-mysql容器的数据挂载 
--需求：查看mysql容器，判断是否有数据卷挂载
--需求：基于宿主机目录实现MySQL数据目录、配置文件、初始化脚本的挂载（查阅官方镜像文档)
---挂载/root/mysql/data到容器内的/var/lib/mysql目录
---挂载/root/mysql/init到容器内的/docker-entrypoint-initdb.d目录，携带课前资料准备的SQL脚本
---挂载/root/mysql/conf到容器内的/etc/mysql/conf.d目录

---在执行docker run命令时，使用-v本地目录:容器内目录可以完成本地目录挂载
---本地目录必须以“I”或".I”开头，如果直接以名称开头，会被识别为数据卷而非本地目录.
-v mysql : /var/lib/mysql 会被识别为一个数据卷叫mysql
-v ./mysql : /var/lib/mysql会被识别为当前目录下的mysql目录

---- 分析
---- 先查看容器挂载
docker volume inspect mysql mysql容器的数据挂载
docker volume inspect nginx nginx 容器的数据挂载
docker volume ls 查看docker所有的挂载
----mysql是隐藏式挂载，会出现匿名，因此可以直接挂载到指定目录

--- 答案
---在home/zsl目录下创建需要要的所有目录
zsl@zsl-virtual-machine:~$ pwd
/home/zsl
zsl@zsl-virtual-machine:~$ mkdir mysql
zsl@zsl-virtual-machine:~$ cd mysql
zsl@zsl-virtual-machine:~/mysql$ pwd
/home/zsl/mysql
zsl@zsl-virtual-machine:~/mysql$ mkdir data
zsl@zsl-virtual-machine:~/mysql$ mkdir conf
zsl@zsl-virtual-machine:~/mysql$ mkdir init

----将下载好的两个文件分别存入创建的文件夹
hm.cnf放置在conf文件夹中
hmall.sql放置在init文件夹中

----创建mysql
docker run -d \
  --name mysql \
  -p 3306:3306 \
  -e TZ=Asia/Shanghai \
  -e MYSQL_ROOT_PASSWORD=123456 \
  -v /home/zsl/mysql/data:/var/lib/mysql \
  -v /home/zsl/mysql/init:/docker-entrypoint-initdb.d \
  -v /home/zsl/mysql/conf:/etc/mysql/conf.d \
  mysql
---此时即可发现在三个文件夹中均有对应的文件产生，则挂载正确
```

# 第三部分 自定义镜像
```shell
镜像就是包含了应用程序、程序运行的系统函数库、运行配置等文件的文件包。构建镜像的过程其实就是把上述文件打包的过程。
层Layer：添加安装包、依赖、配置等，每次操作都形成新的一层。
基础镜像(Baselmage)：应用依赖的系统函数库、环境、配置、文件等。
入口(Entrypoint)：镜像运行入口，一般是程序启动的脚本和参数。

Dockerfile:Dockerfile就是一个文本文件，其中包含一个个的指令(Instruction)，用指令来说明要执行什么操作来构建镜像。将来Docker可以根据Dockerfile帮我们构建镜像.

docker load -i jdk.tar##将压缩包导入

cd demo/##进入到demo文件夹
docker build -t docker-demo .  ##构建镜像
第一步基础镜像
第二步配时区拷贝jar包
第三步入口
---dis##查看镜像
---docker run -d name dd -p 8080:8080 docker-demo###运行
--- dps 检查状态
--- docker logs -f dd##查看日志
```

# 第四部分 网络
```shell
默认情况下，所有容器都是以bridge方式连接到Docker的一个虚拟网桥上。
docker容器虽然都是独立的，但是他们的网关是有联系的
---如dd和mysql，可以先docker exec -it dd bash进入到dd容器，然后通过ping IPAdress可以连通，但是如果IP改变，就链接不成功，因此可以给固定分配。
步骤：
ip addr 查看所有IP以及docker桥
docker network ls
 ---对已有的容器进行连接
 1.创建网络docker network create heima
 2.将容器连接到网络docker network connect heima mysql##网络名在前，容器名在后
 3.通过docker inspect mysql可查看，此时有两个网桥，一个默认的一个heima的
 ---对未创建的容器进行连接
1.docker run -d --name dd -p 8080:8080 --network heima docker-demo
2.docker inspect dd此时只有一个网桥
3.docker exec -it dd bash
4.ping mysql因为它们都属于heima这个网关，因此可以直接通过容器名连接
```

# 项目实例
```shell
-- 部署java应用
重要步骤：准备java的
项目打包，得到架包.jar，与Dockerfile文件一起放入到虚拟机，然后根据指令构建镜像,最后利用docker run部署应用

1.在当前目录下home/zsl创建hmall镜像docker build -t hmall .
2.查看所有镜像docker images
3.docker images
4.docker ps -a
5.docker run -d --name hm -p 8080:8080 --network heima hmall
6.docker logs -f hm
```

# 项目实例
```shell
---部署前端
重要步骤：准备前端nginx需要的文件：html文件夹以及nginx.conf
配置静态资源，项目挂载

1.创建nginx，实现端口映射
docker run -d \
  --name nginx \
  -p 18080:18080 \
  -p 18081:18081 \
  -v /home/zsl/nginx/html:/usr/share/nginx/html \
  -v /home/zsl/nginx/nginx.conf:/etc/nginx/nginx.conf \
  -- network heima \
  nginx
2.docker ps -ad
3.docker logs -f hm即可查看得到
```

# 项目实例
```shell
----分散部署没有完整性，可利用DockerCompose
---Docker Compose通过一个单独的docker-compose.yml模板文件（YAML格式）来定义一组相关联的应用容器，帮助我们实现多个互相关联的docker容器的快速部署。

dockercompose一键部署：
首先清除所有镜像所有容器
docker rm -f nginx hm mysql/
docker rmi hmall docker-demo
docker compose up -d 利用compose一件部署 (-d代表后台运行)
docker compose ps 查看项目的所有进程
docker compose down 移除所有容器和镜像
```


