# Kubernetes

> [kubernetes官网](https://kubernetes.io/docs/home/)

Kubernets是Google在2014年开源出来的。Kubernetes是一个开源容器编排引擎，用于容器化应用的自动化部署，扩展和管理。

## 1. 它能做什么？

* 快速、可预测地部署应用
* 即时扩展应用的能力
* 不影响业务的情况下，无缝发布新功能
* 优化硬件资源，降低成本

## 一些概念

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

# clean up
kubectl delete service hello-node
kubectl delete deployment hello-node
# optionlly, stop minikube virtual machine
minikube stop
# optionlly, delete minikube VM
minikube delete
~~~

## 2. 快速上手

## 安装和设置kubectl

`kubectl`是Kubernetes的命令行工具，让你可以对Kubernetes集群运行命令。你可以使用kubectl部署应用、查看和管理集群资源以及查看日志。

> 安装kubectl时，需要注意版本。kubectl与Kubernetes集群的版本最小差异控制在1个版本以内。举例来说，客户端版本是1.2，那么它可以与1.1、1.2、1.3的master交互。

[安装kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/#)

## 安装Minikube

Minukube是单节点的Kubernetes集群工具，运行在你个人计算机的虚拟机里。

[安装文档，请根据个人计算机的系统进行安装](https://kubernetes.io/docs/tasks/tools/install-minikube/)

