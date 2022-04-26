# Docker 常用命令

## 1. 镜像相关

搜索镜像仓库中的镜像
```
docker search <image>
```
从镜像仓库下载导本地
```
docker pull <image>
```
列出本地已下载的镜像
```
docker images
```
删除本地镜像
```
docker rmi <image>
```

## 2. 容器相关
新建并启动容器
```
docker run
    -d 选项：后台运⾏
    -e 选项：设置环境变量
    -p 选项：端口映射（宿主端⼝:容器端⼝）
    --name 选项：指定容器名称
    --link 选项：链接不同容器
    -v 选项：目录映射（宿主⽬录:容器⽬录）
```

停止容器
```
docker stop <容器名>
```
删除已停止的容器
```
docker rm <容器名>
```
列出运行中的容器
```
docker ps
```
查看运行中的容器
```
docker logs <容器名>
```

