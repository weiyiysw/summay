# Kubernetes

> [kubernetes官网](https://kubernetes.io/docs/home/)

Kubernets是Google在2014年开源出来的。Kubernetes是一个开源容器编排引擎，用于容器化应用的自动化部署，扩展和管理。

## 1. 它能做什么？

* 快速、可预测地部署应用
* 即时扩展应用的能力
* 不影响业务的情况下，无缝发布新功能
* 优化硬件资源，降低成本

## 2. 快速开始

### 安装和设置kubectl

`kubectl`是Kubernetes的命令行工具，让你可以对Kubernetes集群运行命令。你可以使用kubectl部署应用、查看和管理集群资源以及查看日志。

> 安装kubectl时，需要注意版本。kubectl与Kubernetes集群的版本最小差异控制在1个版本以内。举例来说，客户端版本是1.2，那么它可以与1.1、1.2、1.3的master交互。

[安装kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/#)

### 安装Minikube

Minukube是单节点的Kubernetes集群工具，运行在你个人计算机的虚拟机里。

[安装文档，请根据个人计算机的系统进行安装](https://kubernetes.io/docs/tasks/tools/install-minikube/)

### 一些概念

* Kubernetes Pod：包含一个或多个容器的组
* Kubernetes Deployment：对Pod进行健康检查；重启Pod的容器，当它终止时。它是管理创建和扩展Pod的推荐方式。
* Kubernetes Service：通过Kubernetes的虚拟网络，将Pod暴露出来，以便外部可以访问。

~~~shell
# create Deployment
kubectl create deployment hello-node --image=gcr.io/hello-minikube-zero-install/hello-node

# view the Deployment
kubectl get deployments

# view the Pod
kubectl get Pods

# expose the pod to the public internet
# --type=LoadBalancer 表明你想把集群服务暴露出来
kubectl expose deployment hello-node --type=LoadBalancer --port=8080

# scale
kubectl scale deployments/hello-node --replicas=2

# clean up
kubectl delete service hello-node
kubectl delete deployment hello-node

# optionlly, stop minikube virtual machine
minikube stop
# optionlly, delete minikube VM
minikube delete
e0cad8c7-c1de-303f-8599-042d192facd4
~~~

## 安装Kubernetes——本步骤只在CentOS7

### 0. 安装前准备

* 3台centos7机器
* 192.168.56.101 centos7-kube-01
* 192.168.56.102 centos7-kube-02
* 192.168.56.103 centos7-kube-03
* root权限

建议升级内核，默认CentOS 7内核版本3.10.x 内核存在bug，导致docker、Kubernetes不稳定。

~~~shell
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

`vi /etc/hosts`在每台机器上都配置好hosts：

~~~shell
192.168.56.101 centos7-kube-01
192.168.56.102 centos7-kube-02
192.168.56.103 centos7-kube-03
~~~

如果你是在虚拟机里测试，建议可关闭三台机器的防火墙：

~~~shell
sudo sysetmctl stop firewalld.service
~~~

如果不想关闭防火墙，那么按照要求，开放端口

~~~shell
# zone: 作用域
# $port: 替换为具体的端口号，格式为：端口/通讯协议
# permanent: 永久生效
sudo firewall-cmd --zone=public --add-port=$port/tcp --permanent
~~~

端口如下：

控制节点：

| Protocol | Direction | Port Range | Purpose                 | Used By              |
| -------- | --------- | ---------- | ----------------------- | -------------------- |
| TCP      | Inbound   | 6443       | Kubernetes API server   | ALL                  |
| TCP      | Inbound   | 2379-2380  | etcd server client API  | kube-apiserver, etcd |
| TCP      | Inbound   | 10250      | Kubelet API             | Self, Control plane  |
| TCP      | Inbound   | 10251      | kube-scheduler          | Self                 |
| TCP      | Inbooud   | 10252      | kube-controller-manager | Self                 |

工作节点：

| Protocol | Direction | Port Range  | Purpose             | Used By             |
| -------- | --------- | ----------- | ------------------- | ------------------- |
| TCP      | Inbound   | 10250       | Kubelet API         | Self, Control plane |
| TCP      | Inbound   | 30000-32767 | NodePort Services** | All                 |

### 1. 安装docker

~~~shell
# Install Docker CE
## Set up the repository
### Install required packages.
yum install yum-utils device-mapper-persistent-data lvm2

### Add Docker repository.
yum-config-manager \
  --add-repo \
  https://download.docker.com/linux/centos/docker-ce.repo

## Install Docker CE.
yum install docker-ce docker-ce-cli containerd.io

## Create /etc/docker directory.
mkdir /etc/docker

# Setup daemon.
# 启动kubernetes时，cgroupdriver需要时systemd，因此这一步不能省略
# 如果省略了，那么在安装Kubernetes后，需要重新配置，并重启docker
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
EOF

# start docker
sudo systemctl start docker
# ---options---
# create docker group
sudo groupadd docker

# add your user to the docker group
sudo usermod -aG docker $user

# Configure Docker start on boot
sudo systemctl enable docker

# when change daemon.json
# reload systemctl configuration 
sudo systemctl daemon-reload
# restart docker
sudo systemctl restart docker.service
~~~

### 2. 禁用SELinux

~~~shell
setenforce 0
sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
~~~

### 3. 启用br_netfilter内核模块

~~~shell
modprobe br_netfilter
echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables
echo '1' > /proc/sys/net/bridge/bridge-nf-call-ip6tables

# 在 /etc/sysctl.conf添加这两行
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
# 执行此命令，使其生效
sysctl -p
~~~

### 4. 禁用SWAP空间

~~~shell
swapoff -a
# close swap on boot
# open fstab, then comment the swap line UUID
vim /etc/fstab
~~~

### 5. 安装Kubernetes

~~~shell
# set repo
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

# install kubernetes
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

# start kubernetes
systemctl start kubelet

# on boot
systemctl enable kubelet
~~~

### 6. 初始化Kubernetes集群

~~~shell
# 使用 kubeadm init初始化集群
# 192.168.56.101是我虚拟机配置的IP，需替换
kubeadm init --apiserver-advertise-address=192.168.56.101 --pod-network-cidr=10.244.0.0/16
~~~

> [kubeadm init参数详解](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm/)

等待执行完成。执行完成后，将这一句拷贝出来，后续有用。

~~~shell
kubeadm join 192.168.56.101:6443 --token xgrxz9.wu2kx3dbnzbj9q4g \
    --discovery-token-ca-cert-hash sha256:9cf3879f6f9e04341c30bae61ce38de6027381dcbb2d58388c4caa7ac4eeb06e
~~~

以root用户执行：

~~~shell
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
~~~

或者可执行`export KUBECONFIG=/etc/kubernetes/admin.conf`

然后下载`Flannel`

~~~shell
# 下载flannel文件
curl -O https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
# 检查文件里是否含有 iface 配置，默认下载的是没有的，需要手动添加上该配置
# 如果你搭建k8s的VM只有一个网卡，可以不用配置。
# 如果你搭建k8s的VM和我一样，一个NAT网卡用于上网，一个host-only用于组网，那么配置该参数，指定--iface
# 注意，里面涉及该参数的有好几个地方，都需要添加
# 如果不指定的话，会导致不同node之间的Pods无法通信。
container:
    ......
    command：
    -  /opt/bin/flanneld
    arg:
    - --ip-masq
    - --kube-subnet-mgr
    - --iface=eth0


# 启用fannel
kubectl apply -f kube-flannel.yml
~~~

![flannel网络架构图](images\flannel-architecture.png)

> [flannel使用遇到的问题可查看此URL](https://www.codercto.com/a/37238.html)

切换到非root用户，执行

~~~shell
# 执行以下指令后，可以使用非root用户使用指令
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
~~~

### 7. 将另外两台机器加入到集群

在另外两台机器上，执行`kubeadm init`完成后的`kubeadm join`指令

~~~shell
kubeadm join 192.168.56.101:6443 --token xgrxz9.wu2kx3dbnzbj9q4g \
    --discovery-token-ca-cert-hash sha256:9cf3879f6f9e04341c30bae61ce38de6027381dcbb2d58388c4caa7ac4eeb06e
~~~

执行完成后，可在master机器上查看，可以看到另两台机器加入到了集群中

~~~shell
kubectl get nodes
kubectl get pods --all-namespaces
~~~

### 8. 创建Nginx服务

~~~shell
# 创建 deployment
kubectl create deployment nginx --image=nginx

# 将nginx服务暴露出来，以便外部可以访问
kubectl expose deployment nginx --type=LoadBalancer --port=80 --external-ip=192.168.56.104

# 查看服务
kubectl get svc
# demo 显示的结果
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP      10.96.0.1       <none>        443/TCP        112m
nginx        LoadBalancer   10.111.55.104   192.168.56.104     80:30223/TCP   42s

# 查看服务是在哪个节点部署的
kubectl get pods -o wide

# 检验是否启动成功，我这里测试时默认在kube-02机器上
curl centos7-kube-02:30223

# 扩展服务数量
kubectl scale deploy nginx --replicas=2

# 查看服务是在哪个节点部署的，即可看到正在新增节点
kubectl get pods -o wide
~~~

## Kubernetes基础

### Kubernetes Cluster

Kubernetes协调一个高可用的集群，集群里的每台计算机都是一个工作单元。抽象的说，Kubernetes允许你部署一个容器化的应用到集群中，而不是在单个的机器上一个个的执行部署。为使用这种新的部署模型，应用需要以与独立的主机解耦的方式打包：它们需要被容器化。与过去的部署模型相比，容器化应用程序更加灵活和可用。过去部署模型，应用程序直接安装到特定计算机上，因为程序包深度集成到主机中。Kubernetes以一种更有效的方式在集群中自动化分发、调动容器应用。

Kubernetes Cluster包含两类资源：

* Master协调集群
* Node：工作节点，运行应用。

![Kuberneters cluster图](./images/module_01_cluster.svg)

Master负责管理集群。master协调集群内所有的活动，比如：调度应用、保持应用所需状态、缩放应用、滚动更新。

Node可以是一个虚拟机或物理机，它作为Kubernetes cluster里的工作机器。每个节点都有一个Kubelet，作为一个代理（agent）用于管理节点并和Kubernetes master通信。它也应该有处理容器操作的工具，如：Docker或rkt。一个Kubernetes cluster应该至少有3个节点以便处理生产环境的流量。

当你部署在KUbernetes上部署一个应用，你告诉master启动应用容器。master调度这个容器到集群的节点中。节点使用master服务器公开的Kubernetes API与其通信。终了，用户也可以使用Kubernetes API直接与集群交互。

### Kubernetes Deployments

一旦你有了一个运行的Kubernetes Cluster，你可以在上面部署你的容器化应用了。为了做这个，你创建一个Kubernetes Deployment配置。这个Deployment指示Kubernetes如何创建和更新你应用实例。一旦你创建了一个Deployment，Kubernetes master调度里面提到的应用到集群里独立的节点上。

一旦应用被创建了，Kubernetes Deployment Controller持续监控这些实例。如果这个节点托管的实例关闭或被删除了，Deployment Controller将会用另一个节点的实例来替换这个实例。这提供了一种自我修复机制来解决机器故障或维护问题。

在预编排世界中，安装脚本通常用于启动应用程序，但它们不允许从机器故障中恢复。 通过创建应用程序实例并使它们在节点之间运行，Kubernetes Deployments提供了一种根本不同的应用程序管理方法。

### 公开应用

Kuberneters是会死的。Pods事实上有生命周期的。 当一个工作节点死了，运行在这个节点上的Pod也丢失了。然后，ReplicaSet可以通过创建新Pod来动态地将群集驱动回所需状态，以使应用程序保持运行。作为另一个示例，考虑具有3个副本的图像处理后端。 那些副本是可以交换的; 前端系统不应该关心后端副本，即使Pod丢失并重新创建。这意味着，Kubernetes cluster里的每个Pod都有一个独立的IP地址，即时Pod在相同的节点上，所以需要一种方法来自动协调Pod之间的更改，以便您的应用程序继续运行。

在Kubernetes里，Service是一个抽象。它定义了一组逻辑Pods和一个访问它们的策略。Services允许从属Pod之间的松散耦合。Service可使用YAML（推荐）或JSON来定义，就像其他的Kubernetes对象。服务所针对的Pod集通常由LabelSelector确定。

尽管每一个Pod都有一个独有的IP地址，没有一个Service的话，这些IP不会公开到集群外部。Service允许你的应用接受流量。Service可以用以下几种方式公开，通过在服务配置里指定`type`：

* ClusterIP（默认）：在集群里的内部IP上公开服务。这类型使服务仅能从集群内访问
* NodePort：在集群里每一个被选择的节点的相同端口上使用NAT公开服务。这使的服务可以从集群外使用`<NodeIP>:<NodePort>`访问。ClusterIP的超集。
* LoadBalance：在当前云中创建外部负载均衡器（如果支持），并为服务分配固定的外部IP。NodePort的超集。
* ExternalName：通过返回带有名称的CNAME记录，使用任意名称（在规范中由externalName指定）公开服务。没有使用代理。这个类型需要V1.7或者更高的`kube-dns`。

> external ip在AWS等云上使用，因为那会有VIP，同时有负载均衡。如果是自己测试搭建，那么需要自己搭建负载均衡，将设置的external ip负载均衡到后端的NodeIP上。也就是通过 Ingress 公开服务。

### 伸缩应用

### 更新应用



---

# K8s组件

## Master组件（components，复数）

master组件（复数）提供集群的控制平面。master组件对集群做全局决策（例如：调度），并且检测和响应集群时间（例如：启动一个新的Pod，当部署的副本不够时）。

master组件可以运行在集群里的任意机器上。然而，为了简单，安装脚本通常在同一台机器启动全部的master组件，在这台机器上并且不会运行用户容器。[构建高可用集群](https://kubernetes.io/docs/admin/high-availability/)，安装多个master机器实例。

### kube-apiserver

master上的组件，公开Kuberneters API。它是Kubernetes控制平面的前端。它被设计为水平扩展的，这意味着可以扩展部署到多个实例。

### etcd

一致性与高可用的键值存储，用作Kubernetes所有集群数据的后端存储。如果你的Kubernetes集群使用etcd作为后端存储，确保为这些数据你有备份计划。更详细的信息查阅[官方文档](https://etcd.io/docs/)。

### kube-scheduler

master上的组件，观察新创建的且未分配node的Pods，并且为Pods选择一个新的node运行。

调度决策所考虑的因素包括个人和集体资源需求，硬件/软件/策略约束，近似和非近似规范，数据位置，工作负载间干扰和最后期限

### kube-controller-manager

master上的组件，运行控制器。

逻辑上，一个控制器是一个单独的进程，为减少复杂度，他们被编译成一个二进制文件并且在一个进程中运行。

控制器包括：

* 节点控制器：当节点宕机时，负责通知和响应。
* 副本控制器：负责为系统里的每一个副本控制对象维护正确数量的Pods。
* 端点控制器：填充端点对象（就是加入服务或Pods）.
* 服务账号与Token控制器：为新的命名空间创建默认账户及API访问token。

### cloud-controller-manager

云控制管理运行的控制器可以与底层的云服务商交互。云控制管理二进制文件在Kubernetes1.6正式版里是一个alpha特性。云控制管理仅运行云服务商特定的控制器循环。你必须在kube-controller-manager上禁止控制循环。当启动kube-controller-manager时，你可以禁用这个控制循环，通过设置`--cloud-provider`标志到`extenal`。

云控制管理允许云供应商和Kubernetes的代码独立迭代。先前的版本里，Kubernetes的核心代码依赖于特定的云提供商的功能代码。在未来的版本里，云服务商特定的代码由他们自己维护，当运行Kubernetes时链接到云控制管理。

以下四个controller有云供应商依赖：

* Node Controller：帮助云提供商检查决定一个节点停止响应时是否被删除了。
* Route Controller：在云基础设施上设置路由。
* Service Controller：创建、更新和删除云供应商的负载均衡。
* Volume Controller：创建、链接和挂载卷，并且和云提供商外部的卷交互。

## Node组件（components，复数）

node组件运行在每个节点上，维护运行Pods和提供Kubernetes运行时环境。

### kubelet

运行在集群节点上的一个代理。它确保容器在一个pod里运行。

kubelet接收一组PodSpecs，PodSpecs提供通过多种机制和确保容器像在PodSpecs里描述的健康运行。kubelet不管理不是由Kubernetes创建的容器。

### kube-proxy

`kube-proxy`是一个网络代理运行在集群里的每个节点上。

它通过维护主机上的网络规则并执行连接转发来实现Kubernetes服务抽象。

`kube-proxy`负责请求转发。 `kube-proxy`允许在一组后端函数中进行`TCP`和`UDP`流转发或循环`TCP`和`UDP`转发。

### Container Runtime

Container runtime是一个软件用于运行容器。

Kubernetes支持多种Container Runtimes：Docker、containerd、cri-o、rktlet。

![k8s技术架构图](D:\techDocs\images\k8s-architecture.jpeg)

## 插件（Addons）

### DNS

虽然插件不是严格必须，所有的Kubernetes集群应该要有cluster DNS，因为很多例子依赖它。

集群DNS是一个DNS服务器，它和你环境里的其他DNS服务器不一样，它为Kubernetes服务提供DNS记录服务。

Kubernetes启动的容器会在DNS搜索中自动包含此DNS服务器。

### Web UI (Dashboard)

界面是一个通用目的，基于web UI用于Kubernetes集群。它允许用户管理和解决应用在集群里运行，就像在集群自己一样。

### Contrainer Resource Monitoring



### Cluster-level Logging



----

## kubectl namespace

~~~shell
# create namespace
kubectl create namespace test

# display current namespace
kubectl config view --minify --output 'jsonpath={..namespace}'

# change namespace
kubectl config set-context --current --namespace=test
~~~



# 两种HA架构

* Stacked etcd
* external etcd

![](./images/kubeadm-ha-topology-stacked-etcd.svg)

![](./images/kubeadm-ha-topology-external-etcd.svg)

---

~~~shell
kubeadm init  --kubernetes-version=v1.15.0  --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12 --ignore-preflight-errors=Swap --apiserver-advertise-address=0.0.0.0 --image-repository registry.aliyuncs.com/google_containers

kubeadm join 172.27.0.12:6443 --token bhirgz.2kwhcr4jd3h5dloy \
    --discovery-token-ca-cert-hash sha256:aa9c7ebd9b29c409a861a28d3e8dc9f8ec9c4276338c8590d1894a310675bb6c
~~~

~~~shell
# 运行沙箱
kubectl run busybox -it --image=busybox --restart=Never --rm

~~~

~~~shell
# centos
# 设置hostname, $name
hostnamectl set-hostname $name

# 安装pip
$ wget https://bootstrap.pypa.io/get-pip.py
$ sudo python get-pip.py
$ pip -V
pip 9.0.1 from /usr/lib/python2.7/site-packages (python 2.7)
~~~

---

# Kubernetes持久化存储

在kubernetes里，POD是有生命周期的。假设运行了一段时间的POD因未知因素而被迫退出，那么K8S会自动重建这个POD，但是重新建立的POD是以镜像的**初始状态**建立的。

如果我们的POD是无状态的（Stateless）服务，这对我们来说是毫无影响的。如果我们的POD是有状态的（Stateful），那对我们的服务来说，这是致命的，因为这无法恢复之前的状态。

> 举例，MySQL就是有状态的服务，如果MySQL的POD退出，那么之前的数据应该保留。如果以初始状态建立新的POD，那么我们将丢失所有的数据，如此，MySQL服务也就没有意义了。

K8S通过卷（Volume）进行持久化存储。卷是POD的组成部分，因此也可以被定义在POD的配置里。卷不是一个单独的K8S对象，不能单独的创建和删除。卷对POD里的所有容器都可用的，但是必须将卷挂载到每个容器才能访问。

## 卷的类型

* emptyDir：简单的空目录用于存储瞬时数据。
* hostPath：将node节点的文件系直接挂载到POD上。
* gitRepo：通过检出git仓库的内容初始化卷。
* nfs：NFS共享网络存储。
* gcePersistentDisk (Google Compute Engine Persistent Disk), awsElasticBlockStore (Amazon Web Services Elastic Block Store Volume), azureDisk
  (Microsoft Azure Disk Volume) ：用于挂载云服务商提供的特定存储。
* cinder, cephfs, iscsi, flocker, glusterfs, quobyte, rbd, flexVolume, vsphereVolume, photonPersistentDisk, scaleIO：用于挂载其他类型的网络存储。
* configMap, secret, downwardAPI：特定的类型的卷，用于给POD公开确定的K8S资源和集群信息。
* persistentVolumeClaim：预先或动态分配持久化存储的方式。

卷的类型有很多，当你需要使用的时候学习即可。一个单一的POD在同一实际可以使用多个不同类型的多个卷。卷也可以用来在多个容器之间共享数据。

## K8S解耦底层存储技术

K8S的基本思想是对应用以及应用的开发者隐藏实际的基础设施。开发者部署他们的应用到K8S中，应该不需要知道底层是使用什么存储技术，同样的，也不需要知道运行POD的是什么样的物理服务器。当开发者需要给他们的应用确定数量的持久化存储，他们只需向K8S请求即可，就像在创建POD的时候请求CPU、内存和其他资源一样。

### Persistent Volume(PV) 和 Persistent Volume Claim PVC

PV和PVC是K8S用来解耦底层存储的技术，PV和PVC是K8S的两个新的资源对象。需要由集群管理员创建。

## NFS

### 安装NFS

~~~shell
> yum install -y nfs-utils rpcbind

> mkdir -p /nfsdata
> echo "/nfsdata *(rw,no_root_squash,no_all_squash,sync)" >> /etc/exports

# 启动
> systemctl start nfs
> systemctl start rpcbind

# 配置开机自启
> systemctl enable nfs

# 测试nfs server，在其他机器上
> showmount -e $nfsIP
~~~

## Persistent Volume（PV）

PV并不属于任何namespace，它属于集群。

创建PV，我们可以定义如下的`mypv.yaml`

~~~yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mypv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: nfs
  nfs:
    path: /nfsdata
    server: 192.168.56.101
~~~

~~~shell
# create pv
> kubectl apply -f mypv.yaml

# pv也是k8s的对象，因此，我们可以查看PV、删除PV等一系列操作
> kubectl get pv
~~~

## Persistent Volume Claim（PVC）

PVC可以属于某一个namespace。

创建一个PVC，我们可以定义如下的`mypvc.yaml`

~~~yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mypvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: nfs
~~~

~~~shell
# 创建pvc，只要你一创建pvc，k8s将会找到一个合适的PV绑定到pvc。
> kubectl apply -f mypvc.yaml

# 查看pvc
> kubectl get pvc
~~~

### POD中使用PVC

创建POD，并使用pvc，我们以yaml形式创建， 如：命名为`demo.yaml`。

~~~yaml
apiVersion: v1
kind: Pod
metadata:
  namespace: test-pvc
  name: test-app
  labels:
    app: daq
    use: api
spec:
  restartPolicy: Always
  containers:
    - name: test-app
      image: (your test app image)
      resources:
        requests:
          cpu: 0.2
          memory: 200Mi
        limits:
          cpu: 0.5
          memory: 500Mi
      ports:
      - containerPort: 8080
      volumeMounts:
      - mountPath: /opt
        name: test-pvc
  volumes:
    - name: test-pvc
      persistentVolumeClaim:
        claimName: mypvc
~~~

~~~shell
# 运行POD
> kubectl apply -f demo.yaml
~~~

----

# StatefulSets：部署多个stateful应用

## 复制有状态的POD

副本集根据单一的POD模板创建多个POD副本。这些副本之间并没有什么区别，除了他们的名字和IP地址。如果POD模板包含特定的PVC关联的卷，副本集里所有的POD都会使用相同的PVC，因此POD都绑定在相同的PV上。

因为对声明的引用位于pod模板中，该模板用于标记多个pod副本，你不能为每一个副本创建独立使用的PVC。你不能使用一个副本集（ReplicaSet）运行每个实例都需要独立存储的分布式数据存储，至少不能使用一个单一的副本集。

## 为每个POD提供一个固定的标识

除了存储，某些集群应用也需要每个实例都具有一个长期存活的稳定标识。随着时间的推移，POD可以被杀死或者被替换为一个新的。当副本集替换一个POD时，新的POD完全是个有新的主机名和IP的新的POD，并且在他们存储卷里的数据可能被POD杀掉。

某些应用拥有一个稳定的网络标识，这个需求在分布式的有状态应用里是非常普遍的。某些应用程序要求管理员在每个成员的配置文件中列出所有其他集群成员及其IP地址（或主机名）。但是在K8S中，每次POD被重新调度，新的POD就会获取新的主机名和IP地址，所以整个应用集群的成员被重新调度时都需要重新去配置。

## StatefulSet

创建一个StatefulSet资源代替ReplicaSet来运行这种类型的POD。StatefulSet是专门针对应用程序的实例必须被视为不可替代的个人的应用程序，每个应用程序具有稳定的名称和状态。

## 节点

~~~shell
# $hostname，是主机名
# master节点当node使用
> kubectl taint node $hostname node-role.kubernetes.io/master-

# master恢复成master only状态
> kubectl taint node $hostname node-role.kubernetes.io/master="":NoSchedule
~~~

