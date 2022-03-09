# yugabyte server for docker-compose

## Set configuration

Specify .env file
```
MASTER_ADDRESSES=${One msster node:ip:port}
MASTER_BROADCAST_ADDRESSES=${current host ip}
TSERVER_BROADCAST_ADDRESSES=${current host ip}
```


## Launch YugabyteDB

First server, launch master & tserver node
```
cat .env
MASTER_ADDRESSES=yb-master-n1:7100:7100
MASTER_BROADCAST_ADDRESSES=10.0.1.100
TSERVER_BROADCAST_ADDRESSES=10.0.1.100
```

```
docker-compose up -d
```

On other servers, only launch tserver node(refer to master node)
```
cat .env
MASTER_ADDRESSES=10.0.1.100:7100
MASTER_BROADCAST_ADDRESSES=10.0.1.110
TSERVER_BROADCAST_ADDRESSES=10.0.1.110
```

```
docker-compose up -d yb-tserver
```

## Show YugabyteDB Dashboard

http://${master node}:7000
