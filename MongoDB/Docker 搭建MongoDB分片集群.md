# Docker 搭建MongoDB分片集群


### 1 搭建config server

启动了3个config server。docker命令如下：

```shell
docker run --name cs0 -itd -v /home/cjj/cute/data/db/mongo/cs0:/data/db -p 27019:27019 mongo --configsvr --replSet "rs_configsvr"  --bind_ip_all
docker run --name cs1 -itd -v /home/cjj/cute/data/db/mongo/cs1:/data/db -p 27029:27019  mongo --configsvr --replSet "rs_configsvr"  --bind_ip_all
docker run --name cs2 -itd -v /home/cjj/cute/data/db/mongo/cs2:/data/db -p 27039:27019  mongo --configsvr --replSet "rs_configsvr"  --bind_ip_all
```

启动完后可以利用`docker ps -a`来看config server是否启动成功。我们通过命令`docker inspect cs0 | grep IPAddress`看下cs0的ip地址，发现地址是172.17.0.2。相同的方法看到cs1和cs2的ip分别为172.17.0.3和172.17.0.4。

`docker exec -it cs0 bash`进入cs0容器里，并连接MongoDB配置服，配置服的端口为27019。命令如下：

```shell
mongo --host 172.17.0.2 --port 27019
```

`docker exec -it cs0 bash`进入cs0容器里，并连接MongoDB配置服，配置服的端口为27019。命令如下：

```shell
mongo --host 172.17.0.2 --port 27019
```

登录进MongoDB后开始配置config server:

```shell
rs.initiate(
  {
    _id: "rs_configsvr",
    configsvr: true,
    members: [
      { _id : 0, host : "172.17.0.2:27019" },
      { _id : 1, host : "172.17.0.3:27019" },
      { _id : 2, host : "172.17.0.4:27019" }
    ]
  }
)
```

返回的ok里的值为1就表示配置成功了。

```shell
{
        "ok" : 1,
        "$gleStats" : {
                "lastOpTime" : Timestamp(1595298634, 1),
                "electionId" : ObjectId("000000000000000000000000")
        },
        "lastCommittedOpTime" : Timestamp(0, 0),
        "$clusterTime" : {
                "clusterTime" : Timestamp(1595298634, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1595298634, 1)
}
```

### 2 搭建分片副本集

启动2组分片副本集，命令如下：

```shell
docker run --name shm00 -itd -v /home/cjj/cute/data/db/mongo/db00:/data/db -p 27018:27018 mongo --shardsvr --replSet "rs_shardsvr0"  --bind_ip_all
docker run --name shm01 -itd -v /home/cjj/cute/data/db/mongo/db01:/data/db -p 27028:27018 mongo --shardsvr --replSet "rs_shardsvr0"  --bind_ip_all
docker run --name shm02 -itd -v /home/cjj/cute/data/db/mongo/db02:/data/db -p 27038:27018 mongo --shardsvr --replSet "rs_shardsvr0"  --bind_ip_all
docker run --name shm10 -itd -v /home/cjj/cute/data/db/mongo/db10:/data/db -p 27048:27018 mongo --shardsvr --replSet "rs_shardsvr1"  --bind_ip_all
docker run --name shm11 -itd -v /home/cjj/cute/data/db/mongo/db11:/data/db -p 27058:27018 mongo --shardsvr --replSet "rs_shardsvr1"  --bind_ip_all
docker run --name shm12 -itd -v /home/cjj/cute/data/db/mongo/db12:/data/db -p 27068:27018 mongo --shardsvr --replSet "rs_shardsvr1"  --bind_ip_all
```

查看shm00和shm10的ip地址：

```shell
docker inspect shm00 | grep IPAddress    #"172.17.0.5"
docker inspect shm10 | grep IPAddress	 #"172.17.0.8"
```

进入shm00容器：

```shell
docker exec -it shm00 bash
mongo --host 172.17.0.5 --port 27018
```

配置副本集rs_shardsvr0：

```shell
rs.initiate(
  {
    _id : "rs_shardsvr0",
    members: [
      { _id : 0, host : "172.17.0.5:27018" },
      { _id : 1, host : "172.17.0.6:27018" },
      { _id : 2, host : "172.17.0.7:27018" }
    ]
  }
)
```

同样，返回的ok里的值为1表示配置成功。

同理配置副本集rs_shardsvr1：

```shell
docker exec -it shm10 bash
mongo --host 172.17.0.8 --port 27018

rs.initiate(
  {
    _id : "rs_shardsvr1",
    members: [
      { _id : 0, host : "172.17.0.8:27018" },
      { _id : 1, host : "172.17.0.9:27018" },
      { _id : 2, host : "172.17.0.10:27018" }
    ]
  }
)
```

### 3 搭建mongos路由器

docker命令如下：

```shell
docker run --name mongos0 -itd -p 27017:27017 --entrypoint "mongos" mongo --configdb rs_configsvr/172.17.0.2:27019,172.17.0.3:27019,172.17.0.4:27019 --bind_ip_all
```

进入容器并连接MongoDB：

```shell
docker exec -it mongos0 bash

mongo --host 172.17.0.11 --port 27017

```

配置分片信息：

```shell
sh.addShard("rs_shardsvr0/172.17.0.5:27018,172.17.0.6:27018,172.17.0.7:27018")

sh.addShard("rs_shardsvr1/172.17.0.8:27018,172.17.0.9:27018,172.17.0.10:27018")

```

至此，MongoDB分片集群就搭建好了。

### 4 集群测试

首先在mongos配置一个database并启动分片：

```shell
sh.enableSharding("mapp")

# 对order集合设置分片规则
sh.shardCollection("mapp.order", {"_id": "hashed" })
```

插入1000条数据：

```sql
use mapp

for (i = 1; i <= 1000; i=i+1){
    db.order.insert({'price': 1})
}

-- 查看数据条数
db.order.find().count()
1000   
```

再到2个分片副本集看下：

```shell
docker exec -it shm00 bash
mongo --host 172.17.0.5 --port 27018

use mapp
db.order.find().count()
507   #有507条数据
```

另一个分片看下：

```shell
docker exec -it shm10 bash
mongo --host 172.17.0.8 --port 27018

use mapp
db.order.find().count()
493		#有493条数据
```

两个分片数据之和就是1000条数据，看来数据是正常的。

接下来，再去副本看下数据：

```shell
docker exec -it shm11 bash
mongo --host 172.17.0.9 --port 27018
rs.slaveOk()	#解决SECONDARY是不允许读写的问题

use mapp
db.order.find().count()
493		#有493条数据
```
以上测试说明MongoDB分片集群没有问题。
