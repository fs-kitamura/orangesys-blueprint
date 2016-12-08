# init orangesys in gke
### create gke in tokyo
>```
gcloud container clusters create orangesys-io \
   --zone asia-northeast1-c \
   --scopes "cloud-platform,storage-ro,logging-write,monitoring-write,service-control,service-management,https://www.googleapis.com/auth/ndev.clouddns.readwrite"
>```

### create namespace
>```
kubectl create namespace opsbot
kubectl create namespace apigateway
>```

### create thrid resource
>```
cat << EOF > storage.yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1beta1
metadata:
  name: generic
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-standard
EOF

kubectl create -f storage.yaml
>```

### create slack8s bot in opsbot namespace
>```
helm install --namespace opsbot --name="k8s-event" --set "SlackToken=xoxb-88888888888-zzzzzzzzzzZossLxVzjZ0koe,SlackChannel=#ops" slack8s
>```

### create kube-cert-manager in apigateway
>```
kubectl create secret --namespace apigateway generic saas-orangesys-io --from-file=orangesys-5f254b751dae.json
helm install --namespace apigateway --name="ssl" kube-cert-manager
>```

### create app.orangesys.io in apigateway
>```
gcloud compute addresses create app-orangesys-io-ingress --global
kubectl create -f ssl-app-orangesys-io-ing.yaml --namespace apigateway
helm install --namespace apigateway --name-template="{{ randAlpha 3 | lower }}" app-orangesys
>```

### create sysapi.orangesys.io in apigateway
>```
gcloud compute addresses create sysapi-orangesys-io-ingress --global

kubectl create -f ssl-sysapi-orangesys-io-ing.yaml --namespace apigateway

helm install --namespace apigateway --name="stripe" --set StripeSecretKey=XXxxxxxxX orangesys-srv

helm install --namespace apigateway --name="sys" --set FIREBASE_URL="https://saas-orangesys-io.firebaseio.com",FIREBASE_SECRETS="XXXXxxxxxx" orangeapi
>```


### create kong in apigateway
>```
helm dep up kong
helm install --namespace apigateway --name="orangesys" kong
>```

### create grafana dashboard storage in apigateway
>```
helm install --namespace apigateway --name="grafana" mariadb
>```

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
