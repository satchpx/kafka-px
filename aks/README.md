# Kafka-px test on AKS

## Pre-requisites
1. Azure account
2. Azure application https://docs.microsoft.com/en-us/azure/active-directory/develop/quickstart-register-app
3. Register application with Azure AD https://docs.microsoft.com/en-us/rest/api/azure/#register-your-client-application-with-azure-ad
4. Note down the `Application ID`, `Tenant ID`, `Object ID` and `Application password`. This will be used in later steps

## Before you begin
```
git clone https://github.com/satchpx/kafka-px
cd kafka-px/aks
touch .creds.env
```

Add the following key-value pairs in `.creds.env`
```
APPID='<Azure application ID>'
TENANTID='<Tenant ID>'
OBJECTID='<Object ID>'
APPPW='<Application password>'
```

## Deploy AKS cluster

```
#./deploy.sh -g sathya-px-es -r eastus2 -c sathya-px-es1 -n 6 -s 1000 -d UltraSSD_LRS
# The above is blocked on enabling UltraSSD in US East 2 for the subscription ID
# For now use Standard SSD
./deploy.sh -g sathya-px-es -r eastus2 -c sathya-px-es1 -n 6 -s 1024 -d Premium_LRS
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
helm  install --name confluent --namespace confluent --values manifests/kafka-values-px-rf3.yaml ../helm-charts/confluent
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
kubectl apply -f ../kafka-simple-producer/kafka-simple-producer.yaml
```
