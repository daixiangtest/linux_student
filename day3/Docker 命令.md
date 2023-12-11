## Docker 命令

（一）docker 基础命令
启动docker

systemctl start docker

关闭docker

systemctl stop docker

重启docker

systemctl restart docker
docker设置随服务启动而自启动

systemctl enable docker
查看docker 运行状态

------如果是在运行中 输入命令后 会看到绿色的active

systemctl status docker
查看docker 版本号信息

docker version
docker info docker 帮助命令

忘记了某些命令便可使用此进行查看与回顾

```shell
docker --help

```

比如 咱忘记了 拉取命令 不知道可以带哪些参数 咱可以这样使用

```shell
docker pull --help
```



（二）docker 镜像命令
查看自己服务器中docker 镜像列表

docker images

搜索镜像

docker search 镜像名
docker search --filter=STARS=9000 mysql 搜索 STARS >9000的 mysql 镜像**拉取镜像** 不加tag(版本号) 即拉取docker仓库中 该镜像的最新版本latest 加:tag 则是拉取指定版本

```shell
docker pull 镜像名 
docker pull 镜像名:tag

```

拉取最新版 mysql

**运行镜像** ----咱拉取一个tomcat 跑起来试一试

```shell
docker run 镜像名
docker run 镜像名:Tag

```

**ex：**

```shell
docker pull tomcat

docker run tomcat
```



一通测试，发现我们拉了好多镜像了，但我们现在根本用不着，这些无用镜像怎么删除呢？

删除镜像 ------当前镜像没有被任何容器使用才可以删除

#删除一个
docker rmi -f 镜像名/镜像ID

#删除多个 其镜像ID或镜像用用空格隔开即可 
docker rmi -f 镜像名/镜像ID 镜像名/镜像ID 镜像名/镜像ID

#删除全部镜像  -a 意思为显示全部, -q 意思为只显示ID
docker rmi -f $(docker images -aq)

强制删除镜像

docker image rm 镜像名称/镜像ID

镜像的基础命令就到这里 下方会使用更复杂的 docker run 命令 来根据镜像启动容器

保存镜像
将我们的镜像 保存为tar 压缩文件 这样方便镜像转移和保存 ,然后 可以在任何一台安装了docker的服务器上 加载这个镜像

命令:

docker save 镜像名/镜像ID -o 镜像保存在哪个位置与名字

exmaple:

docker save tomcat -o /myimg.tar

## docker 容器命令



**查看正在运行容器列表**

```shell
docker ps

```

**查看所有容器** -----包含正在运行 和已停止的

```shell
docker ps -a
```

**容器怎么来呢 可以通过run 镜像 来构建 自己的容器实例**

**运行一个容器**

# -it 表示 与容器进行交互式启动 -d 表示可后台运行容器 （守护式运行）  --name 给要运行的容器 起的名字  /bin/bash  交互路径* docker run -it -d --name 要取的别名 镜像名:Tag /bin/bash 



例如我们要启动一个redis 把它的别名取为redis001 并交互式运行 需要的命令 —我这里指定版本号为5.0.5

```
#1. 拉取redis 镜像
docker pull redis:5.0.5
#2.命令启动
docker run -it -d --name redis001 redis:5.0.5 /bin/bash

```

*#3.查看已运行容器* 

docker ps



## docker-compose



version: '3'

services:

#定义mysql服务

  mysql:

#使用 MySQL 5.6.51 镜像

​    image: mysql:5.6.51

#容器的名称，通过环境变量 CONTAINER_NAME 指定

​    container_name: ${CONTAINER_NAME}

#始终重启容器

​    restart: always

#MySQL 环境变量设置

​    environment:

#设置 MySQL root 用户的密码，通过环境变量 PANEL_DB_ROOT_PASSWORD 指定

​      MYSQL_ROOT_PASSWORD: ${PANEL_DB_ROOT_PASSWORD}

#定义网络连接，这里使用了名为 1panel-network 的外部网络

​    networks:

   1panel-network

#将 MySQL 服务的端口映射到宿主机的指定端口，通过环境变量 PANEL_APP_PORT_HTTP 指定

​    ports:

   ${PANEL_APP_PORT_HTTP}:3306

#定义数据卷，用于持久化 MySQL 数据

​    volumes:

​    ./data/:/var/lib/mysql
​    ./conf/my.cnf:/etc/mysql/my.cnf

​	./log:/var/log/mysql

    # MySQL 启动命令及参数
    #./data/:/var/lib/mysql: 这将本地系统中的 ./data/ 目录与容器中的 /var/lib/mysql 目录进行挂载。这个挂载点用于持久化 MySQL 数据库文件，确保数据在容器重启后仍然可用。
    #./conf/my.cnf:/etc/mysql/my.cnf: 这将本地系统中的 ./conf/my.cnf 文件与容器中的 /etc/mysql/my.cnf 文件进行挂载。这个挂载点用于提供自定义的 MySQL 配置文件，以覆盖默认配置。
    #./log:/var/log/mysql: 这将本地系统中的 ./log 目录与容器中的 /var/log/mysql 目录进行挂载。这个挂载点用于将 MySQL 的日志文件持久化到本地系统，以便于查看和分析 MySQL 的运行日志。
    
    command:
      --character-set-server=utf8mb4
      --collation-server=utf8mb4_general_ci
      --explicit_defaults_for_timestamp=true
      --lower_case_table_names=1
    #--character-set-server=utf8mb4: 设置 MySQL 服务器的字符集为 utf8mb4，这是一种用于支持更广泛字符集的字符编码。  
    # --collation-server=utf8mb4_general_ci: 设置 MySQL 服务器的排序规则为 utf8mb4_general_ci，这是与 utf8mb4 字符集一起使用的常见排序规则。
    #--explicit_defaults_for_timestamp=true: 强制 MySQL 显式设置默认值以符合 SQL 标准，特别是对于 TIMESTAMP 类型的列。
    #--lower_case_table_names=1: 设置 MySQL 表名的大小写规则。在这里，设置为 1 表示将表名视为大小写不敏感。
    # 定义标签，用于标识由哪个应用创建
    labels:
      createdBy: "Apps"
    #labels 部分用于为 Docker 容器添加元数据，以提供有关容器的附加信息。在这里，你为容器添加了一个标签：
    # 定义外部网络 1panel-network
    
    networks:
      1panel-network:
        external: true
    	#networks 部分用于定义 Docker Compose 文件中的网络配置。
    	#-network 是网络的名称，你可以根据需要选择一个有意义的名称。
    	#external: true 表示这是一个外部网络。外部网络通常在 Docker Compose 文件之外创建，然后在需要的服务或容器中引用它。外部网络的目的是允许多个服务或容器之间进行通信，而不必将它们都定义在同一个 Docker Compose 文件中。

#### 这个docker-compos文件如果用命令表示如下：

`docker run -d \`
  `--name ${CONTAINER_NAME} \  # 设置容器的名称，使用环境变量中定义的${CONTAINER_NAME}`
  `--restart always \          # 始终重启容器`
  `-e MYSQL_ROOT_PASSWORD=${PANEL_DB_ROOT_PASSWORD} \  # 设置MySQL root用户的密码`
  `--network 1panel-network \  # 连接到名为1panel-network的外部网络`
  `-p ${PANEL_APP_PORT_HTTP}:3306 \  # 将容器的MySQL服务端口映射到宿主机的指定端口`
  `-v $(pwd)/data:/var/lib/mysql \  # 挂载数据卷，用于持久化MySQL数据`
  `-v $(pwd)/conf/my.cnf:/etc/mysql/my.cnf \  # 挂载配置文件`
  `-v $(pwd)/log:/var/log/mysql \  # 挂载日志目录`
  `mysql:5.6.51 \  # 使用MySQL 5.6.51镜像`
  `--character-set-server=utf8mb4 \  # 设置MySQL服务器的字符集`
  `--collation-server=utf8mb4_general_ci \  # 设置MySQL服务器的排序规则`
  `--explicit_defaults_for_timestamp=true \  # 强制MySQL显式设置默认值以符合SQL标准`
  `--lower_case_table_names=1 \  # 设置表名大小写规则`
  `--label createdBy=Apps  # 添加一个标签，用于标识由哪个应用程序创建`



`-d: 在后台运行容器。`
`--name ${CONTAINER_NAME}: 指定容器的名称，使用Docker Compose文件中定义的${CONTAINER_NAME}环境变量。`
`--restart always: 设置容器始终自动重启。`
`-e MYSQL_ROOT_PASSWORD=${PANEL_DB_ROOT_PASSWORD}: 设置MySQL root用户的密码，使用Docker Compose文件中定义的${PANEL_DB_ROOT_PASSWORD}环境变量。`
`--network 1panel-network: 将容器连接到名为1panel-network的外部网络。`
`-p ${PANEL_APP_PORT_HTTP}:3306: 将容器的MySQL服务端口（3306）映射到宿主机的指定端口，使用Docker Compose文件中定义的${PANEL_APP_PORT_HTTP}环境变量。`
`-v $(pwd)/data:/var/lib/mysql: 挂载数据卷，用于持久化MySQL数据。`
`-v $(pwd)/conf/my.cnf:/etc/mysql/my.cnf: 挂载MySQL配置文件。`
`-v $(pwd)/log:/var/log/mysql: 挂载日志目录。`
`mysql:5.6.51: 使用MySQL 5.6.51镜像。`
`--character-set-server=utf8mb4: 设置MySQL服务器的字符集。`
`--collation-server=utf8mb4_general_ci: 设置MySQL服务器的排序规则。`
`--explicit_defaults_for_timestamp=true: 强制MySQL显式设置默认值以符合SQL标准。`
`--lower_case_table_names=1: 设置表名大小写规则。`
`--label createdBy=Apps: 添加一个标签，用于标识由哪个应用程序创建。`



定义环境变量
`linux`
`export CONTAINER_NAME=my_mysql_container`
`export PANEL_DB_ROOT_PASSWORD=my_secret_password`
`export PANEL_APP_PORT_HTTP=3306`
`windows`
`$env:CONTAINER_NAME = "my_mysql_container"`
`$env:PANEL_DB_ROOT_PASSWORD = "my_secret_password"`
`$env:PANEL_APP_PORT_HTTP = 3306`
`然后，你可以运行之前提到的 docker run 命令，这样这些环境变量的值就会被引用`