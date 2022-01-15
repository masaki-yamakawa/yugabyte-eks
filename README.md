# yugabyte-eks

## Build cli environment

Build cli environment
```
docker-compose build
```

Launch cli environment
```
docker-compose up -d
docker exec -it yugabyte-eks bash
```

## Create EKS cluster

Create EKS cluster
```
eksctl create cluster \
  --name yb-multizone \
  --version 1.21 \
  --region ap-northeast-1 \
  --zones ap-northeast-1a,ap-northeast-1c,ap-northeast-1d \
  --nodegroup-name standard-workers \
  --node-type m5.2xlarge \
  --nodes 3 \
  --nodes-min 1 \
  --nodes-max 4 \
  --managed
```

Create storage class per zone
```
kubectl apply -f eks/storage.yaml
```

## Create YugabyteDB cluster

Add yugabytedb charts repository
```
helm repo add yugabytedb https://charts.yugabyte.com
helm repo update
helm search repo yugabytedb/yugabyte
```

Create YUgabyteDB namespaces
```
kubectl create namespace yb-eks-ap-northeast-1a
kubectl create namespace yb-eks-ap-northeast-1c
kubectl create namespace yb-eks-ap-northeast-1d
```


Install YugabyteDB
```
helm upgrade -i --debug yb-eks-ap-northeast-1a yugabytedb/yugabyte \
 --namespace yb-eks-ap-northeast-1a \
 -f eks/overrides-ap-northeast-1a.yaml --wait

helm upgrade -i --debug yb-eks-ap-northeast-1c yugabytedb/yugabyte \
 --namespace yb-eks-ap-northeast-1c \
 -f eks/overrides-ap-northeast-1c.yaml --wait

helm upgrade -i --debug yb-eks-ap-northeast-1d yugabytedb/yugabyte \
 --namespace yb-eks-ap-northeast-1d \
 -f eks/overrides-ap-northeast-1d.yaml --wait
```

## Check cluster status

```
kubectl get pods --all-namespaces
kubectl get services --all-namespaces
```

## Configure zone-aware replica placement

To make the replica placement zone-aware
```
kubectl exec -it -n yb-eks-ap-northeast-1a yb-master-0 -- bash \
-c "/home/yugabyte/master/bin/yb-admin --master_addresses yb-master-0.yb-masters.yb-eks-ap-northeast-1a.svc.cluster.local:7100,yb-master-0.yb-masters.yb-eks-ap-northeast-1c.svc.cluster.local:7100,yb-master-0.yb-masters.yb-eks-ap-northeast-1d.svc.cluster.local:7100 modify_placement_info aws.ap-northeast-1.ap-northeast-1a,aws.ap-northeast-1.ap-northeast-1c,aws.ap-northeast-1.ap-northeast-1d 3"
```
To see the new configuration, go to http://<external-ip>:7000/cluster-config


## Connect YugabyteDB

To connect and use the YSQL Shell
```
kubectl exec -n yb-eks-ap-northeast-1a -it yb-tserver-0 -- ysqlsh \
  -h yb-tserver-0.yb-tservers.yb-eks-ap-northeast-1a
```

To connect and use the YCQL Shell
```
kubectl exec -n yb-eks-ap-northeast-1a -it yb-tserver-0 -- ycqlsh \
  yb-tserver-0.yb-tservers.yb-eks-ap-northeast-1a
```

To connect and use the psql
```
psql -h <external-ip> -U yugabyte -p 5433 yugabyte
```


## Delete EKS cluster

Delete EKS cluster
```
eksctl delete cluster \
  --name yb-multizone \
  --wait
```
