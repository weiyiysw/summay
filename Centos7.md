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

* 设置时间和时区

  ~~~shell
  # 安装ntp服务包
  sudo yum install ntp
  
  # 设置为开机启动
  sudo systemctl enable ntpd
  
  # 修改启动参数，增加`-g -x参数`,允许ntp服务器在系统时间误差较大时也能工作
  # OPTIONS="-g -x"
  sudo vi /etc/sysconfig/ntpd
  
  # 将系统时区改为上海时间
  sudo ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
  
  # 启动或重启ntp服务
  sudo service ntpd restart
  
  # 查看时间是否正确
  date
  ~~~

  