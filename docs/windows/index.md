防火墙管理命令

```shell
添加规则
netsh advfirewall firewall add rule name="Open Port 8086" dir=in action=allow protocol=TCP localport=8086

这个命令用于在Windows防火墙中创建入站规则，允许TCP协议通过8086端口。具体作用分解：
name="Open Port 8086"：创建名为"Open Port 8086"的规则
dir=in：控制入站流量（从外部进入本机的连接）
action=allow：允许通过
protocol=TCP：针对TCP协议（HTTP/HTTPS使用的协议）
localport=8086：开放指定的端口号

验证规则是否生效
netsh advfirewall firewall show rule name="Open Port 8086"

删除该规则的方法（需要时）
netsh advfirewall firewall delete rule name="Open Port 8086"
```

