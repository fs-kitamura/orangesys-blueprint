# upgrade node cluster with zero downtime
### set container/cluster name
>```
gcloud config set container/cluster orangesysio
>```


### create new node pools
>```
gcloud container node-pools create g1-small-pool \
  --machine-type g1-small --num-nodes=2 --zone=asia-east1-c \
  --scopes="cloud-platform,storage-ro,logging-write,monitoring-write,\
  service-control,service-management,\
  https://www.googleapis.com/auth/ndev.clouddns.readwrite"
>```

### get all nodes
>```
kubectl get nodes
>```

### disable push new pods ,set node is unscheduled
>```
kubectl cordon gke-cluster-1-default-pool-0218fc84-ae26
>```

### remove pod from old node to new node
>```
kubectl drain gke-cluster-1-default-pool-0218fc84-ae26
>```

### delete old node-pools
>```
gcloud container node-pools delete default-pool
>```
