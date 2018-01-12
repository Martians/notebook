[TOC]
<!-- TOC -->

- [功能](#%E5%8A%9F%E8%83%BD)
    - [特点](#%E7%89%B9%E7%82%B9)
    - [场景](#%E5%9C%BA%E6%99%AF)
- [命令](#%E5%91%BD%E4%BB%A4)
    - [用法](#%E7%94%A8%E6%B3%95)
        - [说明](#%E8%AF%B4%E6%98%8E)
        - [常见](#%E5%B8%B8%E8%A7%81)
    - [安装](#%E5%AE%89%E8%A3%85)
    - [命令](#%E5%91%BD%E4%BB%A4)
        - [启动](#%E5%90%AF%E5%8A%A8)
        - [关闭](#%E5%85%B3%E9%97%AD)
        - [状态](#%E7%8A%B6%E6%80%81)
- [配置](#%E9%85%8D%E7%BD%AE)
    - [构建](#%E6%9E%84%E5%BB%BA)
        - [build](#build)
        - [其他](#%E5%85%B6%E4%BB%96)
    - [环境变量](#%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F)
        - [configs](#configs)
    - [服务管理](#%E6%9C%8D%E5%8A%A1%E7%AE%A1%E7%90%86)
    - [系统相关](#%E7%B3%BB%E7%BB%9F%E7%9B%B8%E5%85%B3)
    - [swarm相关](#swarm%E7%9B%B8%E5%85%B3)
        - [deploy](#deploy)
    - [Network](#network)
        - [Service](#service)
    - [Top-level network](#top-level-network)
        - [默认网络](#%E9%BB%98%E8%AE%A4%E7%BD%91%E7%BB%9C)
        - [网络配置](#%E7%BD%91%E7%BB%9C%E9%85%8D%E7%BD%AE)
    - [volumes](#volumes)

<!-- /TOC -->
# 功能
## 特点
* 一个机器上启动多个容器，统一管理
* 与dockerfile配合使用，覆盖、修改其中的配置
* Stream the log output of running services 
* 使用外部定义的宏

## 场景

# 命令

## 用法
### 说明
* DNS，使用同一个网络的容器，可以彼此间访问
  * 通过service名字来访问；通过 [project]_service_1 这个自动生成的容器名来访问
  * 配置link、external_link来访问

### 常见
* 保存volume数据在外边

## 安装 
[官网安装](https://docs.docker.com/compose/install/ "link") 
```
sudo curl -L https://github.com/docker/compose/releases/download/1.18.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

## 命令

### 启动
docker-compose up

    1. 自动启动配置的service；可以指定某一个
    2. 启动配置的所有docker，已启动就忽略
    2. 如果配置发生改变，就stop并start
    3. --scale SERVICE=NUM
    4. -d 后台启动

docker-compose run

    1. 替换配置文件中的执行，进行
    1. 默认配置中的端口不会映射，需要--service-ports，或者自行指定-p
    2. 可以指定执行用户、工作目录；磁盘
    3. 默认会启动service相关的link的容器

docker-compose build

    1. 只进行build

### 关闭
docker-compose down

    1. --rmi type 删除配置中相关的所有images
    2. -v, 删除磁盘
    2. --remove-orphans 删除其他所有容器，除了本地指定的

docker-compose rm

    1. 删除已经停止的容器
    2. -s，强制关闭
    2. -v，删除相关的所有无名volume
    3. 不删除标记为externla的网路

docker-compose pause
docker-compose unpause

### 状态
docker-compose events

docker-compose logs

docker-compose top

docker-compose ps

docker-compose config：检查配置，并展开宏

    1. 执行的命令，当前必须处在compose.yaml文件夹下
    2. 这些命令都是按照固定的格式去管理相关的容器名称；如果改动了工程名，必须在环境变量中更新，否则这些命令无法找到对应的容器


# 配置
[config](https://docs.docker.com/compose/compose-file)

## 构建
### build
* context：指定dockerfile文件位置
* dockerfile：指定dockerfile名称
* args：更新dockerfile中的arg；
    * 用map方式（password: secret）；list方式都可（- buildno=1）
    * 可以写空值，运行时读取环境变量
    ```
    build:
    context: .
    args:
        buildno: 1
        password: secret

    build:
    context: .
    args:
        - buildno=1
        - password=secret
        - username
    ```
* shm_size

### 其他

* container名字不用指定，就是[project]_service_1的名字
  * 如果在其中指定了service对应的容器名，会导致docker-compose logs等命令无法找到容器
```
namenode:
    env_file: ./config.sh
    image: 'uhopper/hadoop-namenode'
    container_name: namenode
    #domainname: data.com
    hostname: namenode
    ports:
      - "8020:8020"
      - '50070:50070'
    volumes:
      - $NAME_HOME:/hadoop/dfs/name
    environment:
      - NOTHING
```
* command：覆盖默认命令
* entrypoint: /code/entrypoint.sh
* expose
* healthcheck
    ```
    healthcheck:
        test: ["CMD", "curl", "-f", "http://localhost"]
        interval: 1m30s
        timeout: 10s
        retries: 3
    ```
* 其他
```
user: postgresql
working_dir: /code

domainname: foo.com
hostname: foo
ipc: host
mac_address: 02:42:ac:11:65:43

privileged: true


read_only: true
shm_size: 64M
stdin_open: true
tty: true
```
## 环境变量
[网上例子](http://blog.csdn.net/ochinchina_cn/article/details/64125216)

**特别说明**
1. env_file中的定义，默认就会使用.env文件，其中定义的宏可以在yml文件中生效
2. env_file中的定义，如果使用其他文件，只能对容器中生效，yml无法访问到这些宏
3. 如果在shell中定义，必须使用export

* env_file
默认访问.env中的变量
```
env_file: .env

env_file:
  - ./common.env
  - ./apps/web.env
  - /opt/secrets.env
```
* environment
```
environment:
  RACK_ENV: development
  SHOW: 'true'
  SESSION_SECRET:

environment:
  - RACK_ENV=development
  - SHOW=true
  - SESSION_SECRET
```

1. sheel 变量

### configs
1. 将config对应的对象，mount到容器中；可以修改对象的目标位置

简写方式
```
version: "3.1"
services:
  redis:
    image: redis:latest
    deploy:
      replicas: 1
    secrets:
      - my_secret
      - my_other_secret
secrets:
  my_secret:
    file: ./my_secret.txt
  my_other_secret:
    external: true
```

1. 外部资源，需要用docker config create进行配置

    完整方式
```
version: "3.1"
services:
  redis:
    image: redis:latest
    deploy:
      replicas: 1
    secrets:
      - source: my_secret
        target: redis_secret
        uid: '103'
        gid: '103'
        mode: 0440
secrets:
  my_secret:
    file: ./my_secret.txt
  my_other_secret:
    external: true
```

## 服务管理
1. depends_on
[start up order](https://docs.docker.com/compose/startup-order/)

    1. 只是等待依赖容器start完毕 
    ```
    version: '3'
    services:
    web:
        build: .
        depends_on:
        - db
        - redis
    redis:
        image: redis
    db:
        image: postgres
    ```
isolation
restart
```
restart: "no"
restart: always
restart: on-failure
restart: unless-stopped
```

## 系统相关
1. logging
    ```
    logging:
    driver: syslog
    options:
        syslog-address: "tcp://192.168.0.42:123"
    ```

1. tmpfs
1. devices
2. sysctls
```
sysctls:
  net.core.somaxconn: 1024
  net.ipv4.tcp_syncookies: 0

sysctls:
  - net.core.somaxconn=1024
  - net.ipv4.tcp_syncookies=0
```

2. unilimt
```
ulimits:
  nproc: 65535
  nofile:
    soft: 20000
    hard: 40000
```
## swarm相关
### deploy

    1. ENDPOINT_MODE
    2. LABELS
    2. PLACEMENT
    3. REPLICAS



## Network

### Service
1. dns、dns_search; 配置一个或者多个
    
    ```
    dns_search: example.com
    dns_search:
    - dc1.example.com
    - dc2.example.com
    ```

2. links: 连接到其他service的容器，对需要连接的容器，定义的额外别名; 
3. external_links：link 本yaml文件之外的容器，针对共享、通用服务

```
dns_search: example.com
dns_search:
  - dc1.example.com
  - dc2.example.com
```

3. extra_hosts：添加到/etc/hosts
```
extra_hosts:
 - "somehost:162.242.195.82"
 - "otherhost:50.31.209.229"
```

4. network_mode：
5. networks：配置service下的网络，Service-level 
```
services:
  some-service:
    networks:
     - some-network
     - other-network
```
6. ALIASES：当前service在对应网路下的别名
7. IP

```
services:
  app:
    image: busybox
    command: ifconfig
    networks:
      app_net:
        ipv4_address: 172.16.238.10
        ipv6_address: 2001:3984:3989::10

networks:
  app_net:
    driver: bridge
    enable_ipv6: true
    ipam:
      driver: default
      config:
      -
        subnet: 172.16.238.0/24
      -
        subnet: 2001:3984:3989::/64
```
7. port
```
ports:
 - "3000"
 - "3000-3005"
 - "8000:8000"
 - "9090-9091:8080-8081"
 - "49100:22"
 - "127.0.0.1:8001:8001"
 - "127.0.0.1:5000-5010:5000-5010"
 - "6060:6060/udp"
```

## Top-level network
   [network](https://docs.docker.com/compose/networking/)

### 默认网络

1. 定义内部网络
```
networks:
  default:
    # Use a custom driver
    driver: custom-driver-1
```

2. 使用已存在的外部网络
```
networks:
  default:
    external:
      name: my-pre-existing-network
```
### 网络配置
1. ipam
```
ipam:
  driver: default
  config:
    - subnet: 172.28.0.0/16
```
1. external
```
version: '2'

services:
  proxy:
    build: ./proxy
    networks:
      - outside
      - default
  app:
    build: ./app
    networks:
      - default

networks:
  outside:
    external: true
```

```
networks:
  outside:
    external:
      name: actual-name-of-network
```

## volumes
1. 多个service公用的磁盘，就使用top-level定义
```
version: "3.2"
services:
  web:
    image: nginx:alpine
    volumes:
      - type: volume
        source: mydata
        target: /data
        volume:
          nocopy: true
      - type: bind
        source: ./static
        target: /opt/app/static

  db:
    image: postgres:latest
    volumes:
      - "/var/run/postgres/postgres.sock:/var/run/postgres/postgres.sock"
      - "dbdata:/var/lib/postgresql/data"

volumes:
  mydata:
  dbdata:
```