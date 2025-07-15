# Docker常用命令

## 一、查询Docker容器日志

### 1.1 基础日志查询命令

```bash
# 查看容器日志
docker logs <容器名或容器ID>

# 实时追踪日志
docker logs -f <容器名或容器ID>

# 查看最后100行日志
docker logs --tail=100 <容器名或容器ID>

# 显示带时间戳的日志
docker logs -t <容器名或容器ID>

# 按时间范围查询
docker logs --since="2023-01-01" --until="2023-01-02" <容器名或容器ID>
```

### 1.2 高级日志管理

#### 直接访问日志文件
```bash
# 日志文件默认路径
/var/lib/docker/containers/<容器ID>/<容器ID>-json.log

# 使用tail命令查看
tail -f /var/lib/docker/containers/<容器ID>/<容器ID>-json.log
```

#### 日志重定向

```bash
# 将日志输出到文件
docker logs <容器名或容器ID> >& container.log
```

### 1.3 日志配置

```bash
# 编辑日志配置
sudo vim /etc/docker/daemon.json
```

添加以下内容：

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

应用配置：

```bash
sudo systemctl restart docker
```

## 二、进入容器控制台

### 2.1 基础进入方法

```bash
# 使用bash进入
docker exec -it <容器名或容器ID> /bin/bash
docker exec -it <容器名或容器ID> bash
# 使用sh进入
docker exec -it <容器名或容器ID> /bin/sh

# 极简容器进入方式
docker exec -it <容器名或容器ID> sh
```

### 2.2 特殊场景处理

```bash
# 附加到运行中容器（谨慎使用）
docker attach <容器名或容器ID>

# 进入已停止的容器
docker start <容器名或容器ID>
docker exec -it <容器名或容器ID> /bin/bash
```
### 常用参数

| 参数 | 说明                    |
| :--- | :---------------------- |
| `-i` | 保持STDIN打开           |
| `-t` | 分配伪终端              |
| `-u` | 指定用户（如`-u root`） |

### 2.3 操作示例

```bash
# 查看运行中容器
docker ps

# 以root身份进入
docker exec -it -u root <容器名或容器ID> /bin/bash
```

### 2.4 退出容器方式

```bash
# 安全退出（保持容器运行）
Ctrl+P → Ctrl+Q

# 常规退出
exit
```

## 三、注意事项

### 日志相关

1. 生产环境务必配置日志轮转
2. json日志文件可能占用大量磁盘空间
3. 容器删除后日志文件会一并删除

### 控制台相关

1. 使用`attach`命令可能导致容器意外停止
2. 极简镜像可能没有bash，需使用sh
3. 退出时注意方式，避免误停容器

### 最佳实践建议

1. 重要容器日志定期备份
2. 进入容器后避免修改重要文件
3. 使用`-u`参数指定合适用户身份操作
