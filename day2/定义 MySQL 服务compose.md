version: '3'

services:
  # 定义 MySQL 服务
  mysql:
    # 使用 MySQL 5.6.51 镜像
    image: mysql:5.6.51
    # 容器的名称，通过环境变量 CONTAINER_NAME 指定
    container_name: ${CONTAINER_NAME}
    # 始终重启容器
    restart: always
    # MySQL 环境变量设置
    environment:
      # 设置 MySQL root 用户的密码，通过环境变量 PANEL_DB_ROOT_PASSWORD 指定
      MYSQL_ROOT_PASSWORD: ${PANEL_DB_ROOT_PASSWORD}
    # 定义网络连接，这里使用了名为 1panel-network 的外部网络
    networks:
      - 1panel-network
    # 将 MySQL 服务的端口映射到宿主机的指定端口，通过环境变量 PANEL_APP_PORT_HTTP 指定
    ports:
      - ${PANEL_APP_PORT_HTTP}:3306
    # 定义数据卷，用于持久化 MySQL 数据
    volumes:
      - ./data/:/var/lib/mysql
      - ./conf/my.cnf:/etc/mysql/my.cnf
      - ./log:/var/log/mysql
	  
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






docker run -d \
  --name ${CONTAINER_NAME} \  # 设置容器的名称，使用环境变量中定义的${CONTAINER_NAME}
  --restart always \          # 始终重启容器
  -e MYSQL_ROOT_PASSWORD=${PANEL_DB_ROOT_PASSWORD} \  # 设置MySQL root用户的密码
  --network 1panel-network \  # 连接到名为1panel-network的外部网络
  -p ${PANEL_APP_PORT_HTTP}:3306 \  # 将容器的MySQL服务端口映射到宿主机的指定端口
  -v $(pwd)/data:/var/lib/mysql \  # 挂载数据卷，用于持久化MySQL数据
  -v $(pwd)/conf/my.cnf:/etc/mysql/my.cnf \  # 挂载配置文件
  -v $(pwd)/log:/var/log/mysql \  # 挂载日志目录
  mysql:5.6.51 \  # 使用MySQL 5.6.51镜像
  --character-set-server=utf8mb4 \  # 设置MySQL服务器的字符集
  --collation-server=utf8mb4_general_ci \  # 设置MySQL服务器的排序规则
  --explicit_defaults_for_timestamp=true \  # 强制MySQL显式设置默认值以符合SQL标准
  --lower_case_table_names=1 \  # 设置表名大小写规则
  --label createdBy=Apps  # 添加一个标签，用于标识由哪个应用程序创建

-d: 在后台运行容器。
--name ${CONTAINER_NAME}: 指定容器的名称，使用Docker Compose文件中定义的${CONTAINER_NAME}环境变量。
--restart always: 设置容器始终自动重启。
-e MYSQL_ROOT_PASSWORD=${PANEL_DB_ROOT_PASSWORD}: 设置MySQL root用户的密码，使用Docker Compose文件中定义的${PANEL_DB_ROOT_PASSWORD}环境变量。
--network 1panel-network: 将容器连接到名为1panel-network的外部网络。
-p ${PANEL_APP_PORT_HTTP}:3306: 将容器的MySQL服务端口（3306）映射到宿主机的指定端口，使用Docker Compose文件中定义的${PANEL_APP_PORT_HTTP}环境变量。
-v $(pwd)/data:/var/lib/mysql: 挂载数据卷，用于持久化MySQL数据。
-v $(pwd)/conf/my.cnf:/etc/mysql/my.cnf: 挂载MySQL配置文件。
-v $(pwd)/log:/var/log/mysql: 挂载日志目录。
mysql:5.6.51: 使用MySQL 5.6.51镜像。
--character-set-server=utf8mb4: 设置MySQL服务器的字符集。
--collation-server=utf8mb4_general_ci: 设置MySQL服务器的排序规则。
--explicit_defaults_for_timestamp=true: 强制MySQL显式设置默认值以符合SQL标准。
--lower_case_table_names=1: 设置表名大小写规则。
--label createdBy=Apps: 添加一个标签，用于标识由哪个应用程序创建。

## 定义环境变量

linux
export CONTAINER_NAME=my_mysql_container
export PANEL_DB_ROOT_PASSWORD=my_secret_password
export PANEL_APP_PORT_HTTP=3306
windows
$env:CONTAINER_NAME = "my_mysql_container"
$env:PANEL_DB_ROOT_PASSWORD = "my_secret_password"
$env:PANEL_APP_PORT_HTTP = 3306
然后，你可以运行之前提到的 docker run 命令，这样这些环境变量的值就会被引用