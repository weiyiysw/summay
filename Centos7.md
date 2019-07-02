# CentOS7命令总结

## 防火墙

* 查看已经开放的端口

  ~~~shell
  sudo firewall-cmd --list-port
  ~~~

* 开启端口

  ~~~shell
  # zone: 作用域
  # $port: 替换为具体的端口号，格式为：端口/通讯协议
  # permanent: 永久生效
  sudo firewall-cmd --zone=public --add-port=$port/tcp --permanent
  ~~~

* 重启防火墙

  ~~~shell
  # 重启
  sudo firewall-cmd --reload
  # 停止
  sudo sysetmctl stop firewalld.service
  # 禁止开机启动
  sudo systemctl disable firewalld.service
  ~~~

  