# Docker部署Pinpoint

## 部署Pinpoint

### CentOS 7

```sh
# CentOS 7安装成功（ubuntu20安装失败，原因待解决）
git clone https://gitee.com/dousx/pinpoint-docker.git
cd pinpoint-docker
# 安装2.1.0版本
git checkout 2.1.0
docker-compose pull && docker-compose up -d
# 手动设置 所有容器随docker启动自启
docker update --restart=always $(docker ps -q)
docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Status}}" -a
```

### ubuntu20

`pinpoint-docker`中每一个镜像都是亲自构建,Ubuntu镜像会构建失败，可以直接利用`pinpoint-docker`项目下的`.env`和`docker-compose.yml`上传到同一个文件夹下，修改`docker-compose.yml`内容然后执行命令

```sh
docker-compose pull && docker-compose up -d
docker update --restart=always $(docker ps -q)
docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Status}}" -a
```



docker-compose.yml其中已经删除了`build`、`pinpoint-quickstart`以及`pinpoint-agent`

```yml
version: "3.6"

services:
  pinpoint-hbase:
    container_name: "${PINPOINT_HBASE_NAME}"
    image: "pinpointdocker/pinpoint-hbase:${PINPOINT_VERSION}"
    networks:
      - pinpoint

    volumes:
      - /home/pinpoint/hbase
      - /home/pinpoint/zookeeper
    expose:
      # HBase Master API port
      - "60000"
      # HBase Master Web UI
      - "16010"
      # Regionserver API port
      - "60020"
      # HBase Regionserver web UI
      - "16030"
    ports:
      - "60000:60000"
      - "16010:16010"
      - "60020:60020"
      - "16030:16030"
    restart: always
    depends_on:
      - zoo1

  pinpoint-mysql:
    container_name: pinpoint-mysql
    restart: always
    image: "pinpointdocker/pinpoint-mysql:${PINPOINT_VERSION}"
    hostname: pinpoint-mysql
    ports:
      - "3306:3306"
    environment:
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
      - MYSQL_USER=${MYSQL_USER}
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
      - MYSQL_DATABASE=${MYSQL_DATABASE}

    volumes:
      - mysql_data:/var/lib/mysql
    networks:
      - pinpoint

  pinpoint-web:
    container_name: "${PINPOINT_WEB_NAME}"
    image: "pinpointdocker/pinpoint-web:${PINPOINT_VERSION}"

    depends_on:
      - pinpoint-hbase
      - pinpoint-mysql
      - zoo1
    restart: always
    expose:
      - "9997"
    ports:
      - "9997:9997"
      - "${SERVER_PORT:-8080}:${SERVER_PORT:-8080}"
    environment:
      - SERVER_PORT=${SERVER_PORT}
      - SPRING_PROFILES_ACTIVE=${SPRING_PROFILES},batch
      - PINPOINT_ZOOKEEPER_ADDRESS=${PINPOINT_ZOOKEEPER_ADDRESS}
      - CLUSTER_ENABLE=${CLUSTER_ENABLE}
      - ADMIN_PASSWORD=${ADMIN_PASSWORD}
      - CONFIG_SENDUSAGE=${CONFIG_SENDUSAGE}
      - LOGGING_LEVEL_ROOT=${WEB_LOGGING_LEVEL_ROOT}
      - CONFIG_SHOW_APPLICATIONSTAT=${CONFIG_SHOW_APPLICATIONSTAT}
      - BATCH_ENABLE=${BATCH_ENABLE}
      - BATCH_SERVER_IP=${BATCH_SERVER_IP}
      - BATCH_FLINK_SERVER=${BATCH_FLINK_SERVER}
      - JDBC_DRIVERCLASSNAME=${JDBC_DRIVERCLASSNAME}
      - JDBC_URL=${JDBC_URL}
      - JDBC_USERNAME=${JDBC_USERNAME}
      - JDBC_PASSWORD=${JDBC_PASSWORD}
      - ALARM_MAIL_SERVER_URL=${ALARM_MAIL_SERVER_URL}
      - ALARM_MAIL_SERVER_PORT=${ALARM_MAIL_SERVER_PORT}
      - ALARM_MAIL_SERVER_USERNAME=${ALARM_MAIL_SERVER_USERNAME}
      - ALARM_MAIL_SERVER_PASSWORD=${ALARM_MAIL_SERVER_PASSWORD}
      - ALARM_MAIL_SENDER_ADDRESS=${ALARM_MAIL_SENDER_ADDRESS}
      - ALARM_MAIL_TRANSPORT_PROTOCOL=${ALARM_MAIL_TRANSPORT_PROTOCOL}
      - ALARM_MAIL_SMTP_PORT=${ALARM_MAIL_SMTP_PORT}
      - ALARM_MAIL_SMTP_AUTH=${ALARM_MAIL_SMTP_AUTH}
      - ALARM_MAIL_SMTP_STARTTLS_ENABLE=${ALARM_MAIL_SMTP_STARTTLS_ENABLE}
      - ALARM_MAIL_SMTP_STARTTLS_REQUIRED=${ALARM_MAIL_SMTP_STARTTLS_REQUIRED}
      - ALARM_MAIL_DEBUG=${ALARM_MAIL_DEBUG}
    links:
      - "pinpoint-mysql:pinpoint-mysql"
    networks:
      - pinpoint

  pinpoint-collector:
    container_name: "${PINPOINT_COLLECTOR_NAME}"
    image: "pinpointdocker/pinpoint-collector:${PINPOINT_VERSION}"

    depends_on:
      - pinpoint-hbase
      - zoo1
    restart: always
    expose:
      - "9991"
      - "9992"
      - "9993"
      - "9994"
      - "9995"
      - "9996"
    ports:
      - "${COLLECTOR_RECEIVER_GRPC_AGENT_PORT:-9991}:9991/tcp"
      - "${COLLECTOR_RECEIVER_GRPC_STAT_PORT:-9992}:9992/tcp"
      - "${COLLECTOR_RECEIVER_GRPC_SPAN_PORT:-9993}:9993/tcp"
      - "${COLLECTOR_RECEIVER_BASE_PORT:-9994}:9994"
      - "${COLLECTOR_RECEIVER_STAT_UDP_PORT:-9995}:9995/tcp"
      - "${COLLECTOR_RECEIVER_SPAN_UDP_PORT:-9996}:9996/tcp"
      - "${COLLECTOR_RECEIVER_STAT_UDP_PORT:-9995}:9995/udp"
      - "${COLLECTOR_RECEIVER_SPAN_UDP_PORT:-9996}:9996/udp"

    networks:
      - pinpoint
    environment:
      - SPRING_PROFILES_ACTIVE=${SPRING_PROFILES}
      - PINPOINT_ZOOKEEPER_ADDRESS=${PINPOINT_ZOOKEEPER_ADDRESS}
      - CLUSTER_ENABLE=${CLUSTER_ENABLE}
      - LOGGING_LEVEL_ROOT=${COLLECTOR_LOGGING_LEVEL_ROOT}
      - FLINK_CLUSTER_ENABLE=${FLINK_CLUSTER_ENABLE}
      - FLINK_CLUSTER_ZOOKEEPER_ADDRESS=${FLINK_CLUSTER_ZOOKEEPER_ADDRESS}

  #zookeepers
  zoo1:
    image: zookeeper:3.4
    restart: always
    hostname: zoo1
    expose:
      - "2181"
      - "2888"
      - "3888"
    ports:
      - "2181"
    environment:
      ZOO_MY_ID: 1
      ZOO_SERVERS: server.1=0.0.0.0:2888:3888 server.2=zoo2:2888:3888 server.3=zoo3:2888:3888
    networks:
      - pinpoint

  zoo2:
    image: zookeeper:3.4
    restart: always
    hostname: zoo2
    expose:
      - "2181"
      - "2888"
      - "3888"
    ports:
      - "2181"
    environment:
      ZOO_MY_ID: 2
      ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=0.0.0.0:2888:3888 server.3=zoo3:2888:3888
    networks:
      - pinpoint

  zoo3:
    image: zookeeper:3.4
    restart: always
    hostname: zoo3
    expose:
      - "2181"
      - "2888"
      - "3888"
    ports:
      - "2181"
    environment:
      ZOO_MY_ID: 3
      ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=zoo2:2888:3888 server.3=0.0.0.0:2888:3888
    networks:
      - pinpoint

  ##flink
  jobmanager:
    container_name: "${PINPOINT_FLINK_NAME}-jobmanager"
    image: flink:1.3.1
    expose:
      - "6123"
    ports:
      - "${FLINK_WEB_PORT:-8081}:8081"
    command: jobmanager
    environment:
      - JOB_MANAGER_RPC_ADDRESS=jobmanager
    networks:
      - pinpoint
    depends_on:
      - zoo1

  taskmanager:
    container_name: "${PINPOINT_FLINK_NAME}-taskmanager"
    image: flink:1.3.1
    expose:
      - "6121"
      - "6122"
      - "19994"
    ports:
      - "6121:6121"
      - "6122:6122"
      - "19994:19994"
    depends_on:
      - zoo1
      - jobmanager
    command: taskmanager
    links:
      - "jobmanager:jobmanager"
    environment:
      - JOB_MANAGER_RPC_ADDRESS=jobmanager
    networks:
      - pinpoint

volumes:
  data-volume:
  mysql_data:

networks:
  pinpoint:
    driver: bridge
```



## 访问

```sh
# WebUI
http://192.168.174.181:8079/
# flink
http://192.168.174.181:8081/
# QuickStart
http://192.168.174.181:8000/
# Hbase
http://192.168.174.181:16010/
```

## 安装pinpoint-agent

> 地址
>
> https://github.com/pinpoint-apm/pinpoint/releases/download/v2.1.0/pinpoint-agent-2.1.0.tar.gz

将压缩包上传到服务器，移动到/opt/pinpoint-agent目录

```sh
/opt/pinpoint-agent # pwd                                                                       
/opt/pinpoint-agent
/opt/pinpoint-agent # ll                                                                       
总用量 172K
drwxr-xr-x. 2 root root  245 1月  11 21:17 boot
-rw-rw-r--. 1  500  500  255 9月   9 2020 build.info
drwxr-xr-x. 2 root root 4.0K 1月  11 21:17 lib
drwxrwxr-x. 2  500  500    6 9月   9 2020 logs
-rw-rw-r--. 1  500  500  65K 9月   9 2020 pinpoint-bootstrap-2.1.0.jar
-rw-rw-r--. 1  500  500  65K 9月   9 2020 pinpoint-bootstrap.jar
-rw-rw-r--. 1  500  500  14K 9月   9 2020 pinpoint-root.config
drwxr-xr-x. 2 root root 4.0K 1月  11 21:17 plugin
drwxrwxr-x. 4  500  500   34 9月   9 2020 profiles
drwxrwxr-x. 2  500  500   28 9月   9 2020 script
drwxr-xr-x. 2 root root   38 1月  11 21:17 tools
-rw-rw-r--. 1  500  500    5 9月   9 2020 VERSION

```

目录结构如下，修改pinpoint.config配置文件（local和release都修改）

```sh
~/pinpoint-agent-2.1.0/profiles # tree                                                        
.
├── local
│   ├── log4j2.xml
│   └── pinpoint.config
└── release
    ├── log4j2.xml
    └── pinpoint.config

```

修改pinpoint ip

```properties
profiler.transport.grpc.collector.ip=192.168.174.181
profiler.collector.ip=192.168.174.181
```

## 监控jar

- `-javaagent`指定`pinpoint-bootstrap-2.1.0.jar`路径
- `-Dpinpoint.agentId`指定agentId（全局唯一）
- `-Dpinpoint.applicationName`指定应用名称

```sh
java -javaagent:/opt/pinpoint-agent/pinpoint-bootstrap-2.1.0.jar -Dpinpoint.agentId=agentId-1  -Dpinpoint.applicationName=performance-1 -jar performance-0.1.jar
```

启动会稍微有点慢，启动完成之后在WebUI上刷新可看到应用信息



## 删除application

`applicationName`参数设置为需要删除的application

```sh
curl --location --request GET 'http://192.168.174.181:8079/admin/removeAgentId.pinpoint?applicationName=performance-1&agentId=agentId-1&password=admin'
```
