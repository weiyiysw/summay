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

