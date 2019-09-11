# Elasticworx on EKS
Running Elasticsearch backed by Portworx on EKS

## Deploy EKS Cluster
```
eksctl create cluster \
--name ${NAME} \
--region ${REGION} \
--zones ${ZONES} \
--nodegroup-name ${NODEGROUP_NAME} \
--node-type ${NODE_TYPE} \
--nodes ${NODE_COUNT} \
--nodes-min ${NODE_COUNT_MAX} \
--nodes-max ${NODE_COUNT_MIN} \
--node-ami ${AMI} \
--ssh-public-key ${KEY}
```
Note: In this experiment, I am deploying a 3-node EKS cluster running across 3 availability zones.

## Install Portworx
### Before you start
Provide the EC2 instance permissions to create/attach/detach/delete EBS volumes. This is documented [here](https://docs.portworx.com/portworx-install-with-kubernetes/cloud/aws/aws-eks/#prepare)


### Install Portworx using cloud-drives
```
kubectl apply -f 'https://install.portworx.com/?mc=false&kbver=1.13.8&b=true&s=%22type%3Dgp2%2Csize%3D150%22&md=type%3Dgp2%2Csize%3D150&c=px-cluster-402edc71-118a-417c-bfdb-6822a6443b4a&eks=true&stork=true&lh=true&st=k8s'
```

## Install Confluent Kafka
### Install helm
```
https://github.com/helm/helm/blob/master/docs/install.md
```

### Initialize helm
```
helm  init
kubectl create serviceaccount --namespace kube-system tiller
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'
```

### Create the required storageClasses
```
kubectl apply -f manifests/portworx-storageclasses.yaml
```

### Deploy Kafka helm chart
```
helm  upgrade --wait --timeout=600 --install --values manifests/kafka-values-px-rf3.yaml kafka ../helm-charts/confluent
```

### Update the service to have it scraped by prometheus
```
kubectl edit svc <>
```
....and add the following annotation under metadata:
```
apiVersion: v1
kind: Service
metadata:
  annotations:
    prometheus.io/scrape: "true"
```


### Install prometheus and grafana for monitoring
```
git clone https://github.com/satchpx/prom-helm.git
cd prom-helm/
helm install --name prometheus --namespace monitoring prometheus --set server.persistentVolume.storageClass=px-db-rf1-sc,server.shedulerName=stork
helm install --name grafana --namespace monitoring grafana --set persistence.enabled=true,persistence.storageClassName=px-db-rf1-sc,schedulerName=stork
```

### Expose loadbalancer service for grafana
```
kubectl apply -f manifests/grafana-lb.yaml -n monitoring
```

### Deploy Kafka Producer to generate random data
```
kubectl apply -f <TBD>
```
