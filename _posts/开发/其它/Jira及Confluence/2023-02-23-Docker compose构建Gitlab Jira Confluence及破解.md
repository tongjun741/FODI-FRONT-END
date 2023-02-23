---
tags:
    - 其它
    - Jira及Confluence
---

这次构建的compose是

- **gitlab**:latest
- **jira**:8.1.0
- **confluence**:7.7.3
- **mysql**:5.7
- **nginx**:latest

生产部署考虑gitlab和mysql独立出去，不和atlassian家的这俩服务放一起。

gitlab也不使用这里的mysql服务，比较独立，很吃内存，mysql可以用rds云数据库。

接下来的工作只是构建和破解，用于测试，实际生产部署则按需修改。

我自己的docker的配置是给了i5的3个核心，8G内存，刚开始只给4G内存gitlab起不来。

实际没有其它容器的情况下docker吃掉了我15个G的内存，而且是编译完容器运行的时候，可能更多的是macos的内存使用机制导致的。

### 准备工作

- 安装docker
- 安装docker-compose
- 下载[atlassian-agent-v1.2.3](https://gitee.com/pengzhile/atlassian-agent/attach_files/283102/download)破解工具
- 创建docker-compose项目

1.创建一个文件夹存放

```
mkdir my_bussniss_service 
```

2.在该目录下创建如下文件夹及文件

```
.
├── confluence
│   ├── Dockerfile ----------------- confluence的dockerfile
│   └── atlassian-agent.jar -------- 下载好的jar包
├── data --------------------------- 挂载的数据卷
│   ├── confluence
│   │   ├── logs
│   │   └── var
│   ├── gitlab
│   │   ├── config
│   │   ├── log
│   │   └── opt
│   ├── jira
│   │   ├── logs
│   │   └── var
│   └── nginx
│   │   ├── conf.d
│   │   ├── logs
│   │   │   ├── access.log --------- 手动创建
│   │   │   └── error.log ---------- 手动创建
│   │   └── nginx.conf
│   └── mysql
│       ├── backup ----------------- 备份位置
│       ├── conf.d
│       │   └── my.cnf ------------- 需要修改的mysql配置
│       ├── data
│       └── init.sql --------------- jira和confluence初始化sql=
├── jira
│   ├── Dockerfile ----------------- jira的dockerfile
│   └── atlassian-agent.jar -------- 下载好的jar包
└── docker-compose.yml ------------- 主要的docker-compose文件
```

### 二、配置Nginx代理

因为走的是docker的network，所以注意转发地址是和docker-compose.yml中的network别名相同，对应server_name的hosts配置这里就不写了。

通过Nginx对外开放一个80做代理，docker的容器不暴露出来。如果不在意这部分，可以抛弃掉Nginx。直接宿主机访问ports映射的端口。

```
...

    server {
        listen 80;
        server_name git.com;
        location / {
            proxy_pass http://gitlab; # network别名
        }
    }

    server {
        listen 80;
        server_name jira.com;
        location / {
            proxy_pass http://jira:8080; # network别名和expose的端口
        }
    }

    server {
        listen 80;
        server_name wiki.com;
        location / {
            proxy_pass http://confluence:8090; # network别名和expose的端口
        }
    }

...
```

### mysql

#### init.sql

注意 1.jira使用utf8mb4，confluence使用utf8 2.mysql5.7+创建外键需要`REFERENCES`权限，5.6不需要

```
CREATE DATABASE confluence CHARACTER SET utf8 COLLATE utf8_bin;
-- mysql5.6，无需REFERENCES权限，使用内网ip地址和安全性高的密码
GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER,INDEX,REFERENCES on confluence.* TO 'confluence'@'%' IDENTIFIED BY '复杂的密码';
flush privileges;

CREATE DATABASE jira CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
-- mysql5.6，无需REFERENCES权限，使用内网ip地址和安全性高的密码
GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER,INDEX,REFERENCES on jira.* TO 'jira'@'%' IDENTIFIED BY '复杂的密码';
flush privileges;
```

#### my.cnf

```
[mysqld]
default-storage-engine=INNODB
character_set_server=utf8mb4
innodb_default_row_format=DYNAMIC
innodb_large_prefix=ON
innodb_file_format=Barracuda
innodb_log_file_size=2G
transaction-isolation=READ-COMMITTED
max_allowed_packet=256M
```

### Jira

#### Dockerfile

这里的内容和Confluence除了目录基本一致。

使用的是[cptactionhank](https://hub.docker.com/r/cptactionhank)这个兄弟在docker hub上的镜像，需要什么版本可以在上面找。

但是不建议修改版本，atlassian家的软件不同版本对环境配置的要求都不一样，涉及到`my.cnf`需要再看官方手册配置。

**修改版本很大可能导致服务启动起来后报错。**

```
FROM cptactionhank/atlassian-jira-software:8.1.0

USER root

# 将代理破解包加入容器
COPY "atlassian-agent.jar" /opt/atlassian/jira/

# 设置启动加载代理包
RUN echo 'export CATALINA_OPTS="-javaagent:/opt/atlassian/jira/atlassian-agent.jar ${CATALINA_OPTS}"' >> /opt/atlassian/jira/bin/setenv.sh
```

### Confluence

#### Dockerfile

同Jira

```
FROM cptactionhank/atlassian-confluence:7.7.3

USER root

# 将代理破解包加入容器
COPY "atlassian-agent.jar" /opt/atlassian/confluence/

# 设置启动加载代理包
RUN echo 'export CATALINA_OPTS="-javaagent:/opt/atlassian/confluence/atlassian-agent.jar ${CATALINA_OPTS}"' >> /opt/atlassian/confluence/bin/setenv.sh
```

### docker-compose.yml

```
version: '3.6'
services:

  gitlab:
    container_name: mbs_gitlab
    image: gitlab/gitlab-ce:latest #建议使用最新版本，gitlab的漏洞都挺严重的，目前情况来看推荐是12.9+
    restart: always
    #hostname: 'localhost'  指定域名，对应下面的external_url，可以省去再做一次代理
    environment:
      GITLAB_OMNIBUS_CONFIG: |
              external_url 'http://127.0.0.1'
              gitlab_rails['time_zone'] = 'Asia/Shanghai' # 这里下面可以接上ssl的配置和邮件服务的配置
    networks:
      mbs-net:
        aliases:
          - gitlab
    expose:
      - 80
    volumes:
      - ./data/gitlab/config:/etc/gitlab/
      - ./data/gitlab/opt:/var/opt/gitlab/
      - ./data/gitlab/log:/var/log/gitlab/

  mysql:
    image: mysql:5.6
    container_name: mbs_mysql
    restart: always
    networks:
      mbs-net:
        aliases:
          - mysql
    expose:
      - 3306
    environment:
      MYSQL_ROOT_PASSWORD: 123456 # mysql初始密码
    volumes:
      - ./data/mysql/data:/var/lib/mysql/
      - ./data/mysql/conf.d/:/etc/mysql/conf.d/
      - ./data/mysql/backup/:/backup/

  jira:
    build: ./jira
    container_name: mbs_jira
    restart: always
    networks:
        mbs-net:
          aliases:
            - jira
    expose:
      - 8080
    volumes:
        - ./data/jira/var/:/var/atlassian/jira/
        - ./data/jira/logs/:/opt/atlassian/jira/logs/

  confluence:
    build: ./confluence
    container_name: mbs_confluence
    restart: always
    networks:
      mbs-net:
        aliases:
          - confluence
    expose:
      - 8090
    volumes:
      - ./data/confluence/var/:/var/atlassian/confluence/
      - ./data/confluence/logs/:/opt/atlassian/confluence/logs/

  nginx:
    container_name: ss_nginx
    image: nginx:latest
    restart: always
    networks:
      mbs-net:
        aliases:
          - nginx
    expose:
      - 80
    ports:
      - 80:80
    volumes:
      - ./data/nginx/conf.d/:/etc/nginx/conf.d
      - ./data/nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./data/nginx/log/error.log:/var/log/nginx/error.log
      - ./data/nginx/log/access.log:/var/log/nginx/access.log

networks:
  mbs-net:
    driver: bridge
```

### 编译

#### 执行编译。

```
docker-compose up --build #cmd+c即退出 docker-compose up --build -d #后台执行 
```

#### 初始化Jira和Confluence数据库和用户权限

1.进入mysql容器

```
docker exec -it mbs_mysql bash 
```

2.连接mysql

```
mysql -uroot -p # 输入密码 
```

3.加载sql

```
source /root/init.sql; 
```

### 破解

建议先执行以下jar包看下可以使用的参数，实际使用主要就是`-p`和`-s`，Jira和Confluence大同小异

```
docker exec mbs_jira java -jar /opt/atlassian/jira/atlassian-agent.jar 
```

#### Jira

```
docker exec mbs_jira java -jar /opt/atlassian/jira/atlassian-agent.jar \
    -p jira \
    -m a@b.com \
    -n name \
    -o http://127.0.0.1:8080 \
    -s ABCD-EFGH-HIJK-LMNO # 替换为页面上显示的server id
```

#### Jira 插件

```
docker exec mbs_jira java -jar /opt/atlassian/jira/atlassian-agent.jar \
    -p com.xiplink.jira.git.jira_git_plugin \ # 替换为插件id
    -m a@b.com \
    -n a@b.com \
    -o http://127.0.0.1:8080 \
    -s ABCD-EFGH-HIJK-LMNO # 替换为页面上显示的server id
```

#### Confluence

```
docker exec mbs_confluence java -jar /opt/atlassian/confluence/atlassian-agent.jar \
    -p conf \
    -m a@b.com \
    -n name \
    -o http://127.0.0.1:8090 \
    -s ABCD-EFGH-HIJK-LMNO # 替换为页面上显示的server id
```

#### Confluence 插件

```
docker exec mbs_confluence java -jar /opt/atlassian/confluence/atlassian-agent.jar \
    -p com.xiplink.jira.git.jira_git_plugin \ # 替换为插件id
    -m a@b.com \
    -n a@b.com \
    -o http://127.0.0.1:8090 \
    -s ABCD-EFGH-HIJK-LMNO # 替换为页面上显示的server id
```

### 常用命令

1.关闭并删除容器重新创建。如果是想重新build，注意删掉对应data下挂载数据卷的目录，下次build会自动创建。

```
docker-compose down 
```

2.只重启/停止/启动容器

```
docker-compose restart/stop/start 
```

### 注意事项

我自己的docker的配置是给了i5的3个核心，8G内存，刚开始只给4G内存gitlab起不来。

实际没有其它容器的情况下docker吃掉了我15个G的内存，而且是编译完容器运行的时候，可能更多的是macos的内存使用机制导致的。

**1. 内存不够会导致gitlab一直报`exit 137`无限重启，加内存即可解决。**

**2. docker-compose输出done之后服务也是没有办法直接访问的。**

**3. Jira会出现启动进度条，Confluence会转圈，Gitlab会报错，等Jira启动完毕了，差不多Confluence和Gitlab也就可以访问了。**

**4. 机器内存和cpu不够不建议一起编译，会起飞。**

> ***转载使用注明出处。原文链接 https://heimo-he.github.io/docker/2020/09/22/docker-compose-gitlab-jira-confluence/\***

[# Docker](https://heimo-he.github.io/tag/#/Docker) [# Gitlab](https://heimo-he.github.io/tag/#/Gitlab) [# Jira](https://heimo-he.github.io/tag/#/Jira) [# Confluence](https://heimo-he.github.io/tag/#/Confluence)