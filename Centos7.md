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
  sudo systemctl stop firewalld.service
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


## 查看内核信息

~~~shell
# 查看全部信息
uname -a
# 查看内核信息
uname -r
# 查看Linux版本号
cat /etc/redhat-release
~~~

## 安装tools

~~~shell
# 安装ifconfig
sudo yum install net-tools
~~~

## 升级内核

~~~shell
# centos7内核是3.10.x，内核存在bug，导致docker、Kubernetes不稳定
# 升级至5.2.x
# step 1 update all packages to the latest version
sudo yum -y update
# install the following package to make installation and updating process fast
sudo yum -y install yum-plugin-fastestmirror

# step 2
cat /etc/redhat-release
cat /etc/os-release

# step 3, add ELRepo Repository
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
yum repolist

# step 4, install kernel
yum --enablerepo=elrepo-kernel install kernel-ml

# step 5 Configure Grub2 CentOS 7
sudo awk -F\' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg
sudo grub2-set-default 0
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
sudo reboot

# step 6, check kernel
uname -msr

# step 7, remove old kernel(optional)
yum install yum-utils
package-cleanup --lodkernels
~~~

## 查看网卡UUID

~~~shell
 # 查看网卡UUID
 nmcli con show
 # 网卡重新生成UUID，$eth 指网卡名称
 uuidgen $eth
~~~

## Vim操作

~~~shell
# 列编辑
# 使用vim打开一个文件
1. normal模式下，按下Crtl+v，进入视图模式
2. 按 hjkl 选择需要列编辑的行与列位置
3. shift+i 或 shift+a (即大写的I或A)，进入编辑模式
4. 输入要插入的内容（此时看到的光标所在行输入）
5. 按Esc退出编辑模式，等1~2s即会看到列编辑完成。
~~~

## 安装expect

~~~shell
# 安装expect, 就可以写脚本登录
yum install -y expect
# demo: 登录腾讯云
# 可以将 bash -c后的语句换成 ssh，即可实现登录服务器
set timeout 10
spawn bash -c "docker login --username=$name ccr.ccs.tencentyun.com"
expect {
  "*Password*" {send "$passwd\r"}
}
interact
~~~

