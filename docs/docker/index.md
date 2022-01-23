# Linux开发环境配置(Docker)

# 解释：

`Docker`中每一个容器都是独立运行的，相当于一个独立的`linux`系统，如果想便捷地修改容器内的文件，我们就需要把容器目录挂载到主机的目录上。容器端口类似，外界无法直接访问容器内部的端口，需要先将容器端口映射到`linux`主机端口上才能访问

<font color='red'>下面命令注意在root用户下运行，避免重复 sudo 省略</font>

```bash
su -root
```
# Docker

## 安装Docker
参考：[Docker 安装文档](https://docs.docker.com/engine/install/centos/)

### 1. 删除老版本

```bash
 sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

### 2. 安装工具包并设置存储库

```bash
sudo yum install -y yum-utils

sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

### 3. 安装docker引擎

```bash
sudo yum install docker-ce docker-ce-cli containerd.io
```

## Docker使用

### 1. 启动docker

```bash
sudo systemctl start docker
```

### 2. 设置开机启动docker

1. 检查docker版本

```bash
docker -v
```

2. 查看docker已有镜像

```bash
sudo docker images
```

3. 设置docker开机启动

```bash
sudo systemctl enable docker
```

### 3. 设置国内镜像仓库

参考：[阿里云镜像加速服务](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors)

```bash
# 创建文件
sudo mkdir -p /etc/docker
# 修改配置, 设置镜像
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://vw9qapdy.mirror.aliyuncs.com"]
}
EOF
# 重启后台线程
sudo systemctl daemon-reload
# 重启docker
sudo systemctl restart docker
```

# MySQL of Docker

## 1. docker安装mysql

```bash
sudo docker pull mysql:5.7
```

## 2. docker启动mysql

```bash
sudo docker run -p 3306:3306 --name mysql \
-v /mydata/mysql/log:/var/log/mysql \
-v /mydata/mysql/data:/var/lib/mysql \
-v /mydata/mysql/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=root \
-d mysql:5.7
```

参数解释：

- -p 3306:3306：将容器的3306端口映射到主机的3306端口
- --name：给容器命名

- -v /mydata/mysql/log:/var/log/mysql：将配置文件挂载到主机/mydata/..
- -e MYSQL_ROOT_PASSWORD=root：初始化root用户的密码为root

查看docker启动的容器:

```bash
docker ps
```

## 3. 配置mysql

1. 进入挂载的mysql配置目录

```bash
cd /mydata/mysql/conf
```

1. 修改配置文件 `my.cnf`

```bash
vi my.cnf
```

copy以下内容：

```bash
[client]
default-character-set=utf8
[mysql]
default-character-set=utf8
[mysqld]
init_connect='SET collation_connection = utf8_unicode_ci'
init_connect='SET NAMES utf8'
character-set-server=utf8
collation-server=utf8_unicode_ci
skip-character-set-client-handshake
skip-name-resolve

# Esc
# :wq
```

1. docker重启mysql使配置生效

```bash
docker restart mysql
```

# Redis of Docker

## 1. docker拉取redis镜像

```bash
docker pull redis
```

## 2. docker启动redis

1. 创建redis配置文件目录

```bash
mkdir -p /mydata/redis/conf

touch /mydata/redis/conf/redis.conf
```

1. 启动redis容器

```bash
docker run -p 6379:6379 --name redis \
-v /mydata/redis/data:/data \
-v /mydata/redis/conf/redis.conf:/etc/redis/redis.conf \
-d redis redis-server /etc/redis/redis.conf
```

## 3. 配置redis持久化

更多redis配置参考：[redis配置](https://raw.githubusercontent.com/redis/redis/6.0/redis.conf)

```bash
echo "appendonly yes"  >> /mydata/redis/conf/redis.conf

# 重启生效
docker restart redis
```

# 容器随docker启动自动运行

```shell
# mysql
docker update mysql --restart=always

# redis
docker update redis --restart=always
```



# 