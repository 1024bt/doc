```shell
#!/bin/bash

# 检查并创建目录
echo "检查并创建所需的目录..."
mkdir -p /mydata/nginx/conf.d
mkdir -p /mydata/redis/conf
mkdir -p /mydata/redis/data
mkdir -p /mydata/mysql/conf
mkdir -p /mydata/mysql/data
mkdir -p /mydata/java
echo "目录创建完成！"

# 获取当前机器的IP地址
IP_ADDRESS=$(hostname -I | awk '{print $1}')

# 创建Nginx反向代理配置文件
echo "正在创建Nginx反向代理配置文件..."
sudo bash -c 'cat > /mydata/nginx/conf.d/default.conf <<EOF
server {
    listen 80;
    server_name '$IP_ADDRESS';
	
	location / {
        root   /usr/share/nginx/html/dist/;
        index  index.html index.htm;
    }
    location /api {
        proxy_pass http://'$IP_ADDRESS':8080/;
        proxy_set_header Host \$host;
        proxy_set_header X-Real-IP \$remote_addr;
    }
}
EOF'
echo "Nginx反向代理配置文件创建完成！"

# 安装Nginx并将80端口映射到宿主机，并挂载Nginx配置文件目录
echo "正在安装Nginx并进行端口映射并挂载Nginx配置文件目录..."
docker run -e TZ="Asia/Shanghai" -d --name nginx --restart always -p 80:80 -v /mydata/nginx/conf.d:/etc/nginx/conf.d -v /mydata/nginx/html/:/usr/share/nginx/html/ nginx
echo "Nginx安装完成！"

# 重启Nginx容器以应用配置更改
echo "重启Nginx容器以应用配置更改..."
docker restart nginx
echo "Nginx重启完成！"

# 安装Redis并启用AOF持久化并将6379端口映射到宿主机
echo "正在安装Redis并启用AOF持久化并进行端口映射..."
docker run -d -e TZ="Asia/Shanghai" --name redis --restart always -v /mydata/redis/data:/data -v /mydata/redis/conf:/usr/local/etc/redis -p 6379:6379 redis:6.2.8 redis-server --appendonly yes
echo "Redis安装完成！"

# 安装MySQL 5.7并挂载数据存储目录并进行端口映射，并挂载MySQL配置文件目录
echo "正在安装MySQL 5.7并挂载数据存储目录并进行端口映射并挂载MySQL配置文件目录..."
docker run -e TZ="Asia/Shanghai" -d --name mysql --restart always -v /mydata/mysql/data:/var/lib/mysql -v /mydata/mysql/conf:/etc/mysql/conf.d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=Bast.2023 mysql:5.7
# MYSQL_ROOT_PASSWORD=Bast.2023 设置数据库root密码
echo "MySQL 5.7安装完成！"


# 安装JDK 1.8并将8080端口映射到宿主机
echo "正在安装JDK 1.8并进行端口映射和运行项目..."
docker run -e TZ="Asia/Shanghai" -d --name jdk8 --restart always -p 8080:8080 -v /mydata/java:/app openjdk:8 sh -c "mkdir -p /app/logs && java -jar /app/test-0.0.1-SNAPSHOT.jar >> /app/logs/application.log"
echo "JDK 1.8安装完成,项目运行成功！"

echo "所有组件安装完成！"

```

