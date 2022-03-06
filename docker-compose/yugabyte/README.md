# yugabyte server for docker-compose

## Set configuration

Specify .env file
```
MASTER_ADDRESSES=${One msster node:ip:port}
MASTER_BROADCAST_ADDRESSES=${current host ip}
TSERVER_BROADCAST_ADDRESSES=${current host ip}
```


## Launch YugabyteDB

Launch master & tserver node
```
docker-compose up -d
```

Launch tserver node(refer first master node)
```
docker-compose up -d yb-tserver
```
