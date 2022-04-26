# 启动MongoDB


##1. 下载镜像
在CMD or PowerShell 使用命令行下载并查看Docker 下的镜像
```
docker pull <image>
docker pull mongo
docker images
```


## 2.创建Volume
创建文件（磁盘卷）持久化数据，否则当MongoDB容器停止时会导致数据丢失
```
docker volume create --name mongodata
```

## 3. 运行Mongo 容器
```
docker run --name mongo -p 27017:27017 -v mongodata:/data/db -e MONGO_INITDB_ROOT_USERNAME=admin -e MONGO_INITDB_ROOT_PASSWORD=admin -d mongo 
```

可用ps命令查看是否启动成功：
```
docker ps
```

## 4.登录 MongoDB
登录到MongoDB容器中：
```
docker exec -it mongo bash
```
然后再mongoDB容器中链接MongoDB:
```
mongo -u admin -p admin
```
## 5. 创建数据库并搭建springboot demo

创建库
```
use springbucks
```
创建具有读写权限的用户
```
db.createUser(
    {
        user: "springbucks",
        pwd: "springbucks",
        roles:[
            {role:"readWrite",db:"springbucks"}
        ]
    }
)
```




