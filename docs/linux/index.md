####  CentOs7关闭防火墙的命令!
1:查看防火状态
	systemctl status firewalld
	service  iptables status

2:暂时关闭防火墙
	systemctl stop firewalld
	service  iptables stop
3:永久关闭防火墙
	systemctl disable firewalld
	chkconfig iptables off

4:重启防火墙
	systemctl enable firewalld
	service iptables restart



#### 添加防火墙规则
```shell
firewall-cmd --list-ports #查看已经开放的端口
firewall-cmd --permanent --zone=public --add-port=端口/tcp #开放端口
firewall-cmd --reload
```



#### 后台挂起脚本
```shell
nohup command &>/dev/null &

通过jobs -l 可查看后台执行的脚本或命令
```



#### 查看端口占用情况
```shell
lsof -i:端口号。lsof 可能需要安装

netstat -tunlp | grep 端口号 
  -t (tcp) 仅显示tcp相关选项
  -u (udp)仅显示udp相关选项
  -n 拒绝显示别名，能显示数字的全部转化为数字
  -l 仅列出在Listen(监听)的服务状态
  -p 显示建立相关链接的程序名
```

##### 查看占用
	htop 按q退出



##### 查看占用

```shell
grep -C 10 '通用异常' spring.log #查询spring.log文件中出现的'通用异常' 显示上下10行

grep -n -C 10 --color=auto '通用异常' spring.log  # 带行号和高亮
```

