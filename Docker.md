# Docker

## 构建镜像

~~~shell
> docker build -t $imageName:$tag .
~~~

## 删除

~~~shell
# 过滤出包含Tencent和test，批量删除docker image
> docker rmi $(docker image ls | grep tencent | grep test | awk '{print $3}')
~~~

## 异常处理

在Docker中，使用jmap -heap pid命令报错。

~~~shell
attaching to process: sun.jvm.hotspot.debugger.DebuggerException: Can't attach to the process: ptrace(PTRACE_ATTACH, ..) failed for 6: Operation not permitted
~~~

经查，是Docker在1.10以后增加了安全特性导致的。

### 解决方案

1. --security-opt seccomp=unconfined

    ~~~shell
    > docker run --security-opt seccomp=unconfined image:tag
    ~~~

2. --cap-add=SYS_PTRACE

   ~~~shell
   > docker run --cap-add=SYS_PTRACE image:tag
   ~~~

3. Docker compose支持

    ~~~shell
    services:
      mysql:
        ...
      api:
        ...
        cap_add:
          - SYS_PTRACE
    ~~~

> 参考文档：[JVM in Docker and PTRACE_ATTACH](https://jarekprzygodzki.wordpress.com/2016/12/19/jvm-in-docker-and-ptrace_attach/)

