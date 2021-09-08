# Monitoring Stack Helm Chart
This helm chart includes 
- Prometheus, Grafana
- Kube-state-metrics, node-exporter
- Event-exporter for k8s events
- Kafka-exporter for kafka metrics
- Prometheus scrape config to scrape spark master

Installation
```
helm install -f values.yaml monitoring .
```

### Things to configure on values.yaml
- Change the storage:className,
you can find your storageclass by running 
```
kubectl get storageclasses
```  

- Change the ingress:classname,
you can find your ingressclass by running
```
kubectl get ingressclasses
```
if nothing found, you can put nginx

- you dont have to change the subdomains 
as these ingresses will create loadbalancer addresses


### Enabling spark metrics
- passing --set metrics.enabled to spark installation is needed
```
helm install spark --set metrics.enabled=true bitnami/spark
```

### Kafka exporter config
- On the kafka-exporter/kafka-exporter-deployment.yaml,
the line 
```
'--kafka.server=kafka:9092'
```
should point the kafka service, change it if necessary. 
Probably you won't need to change it as it's the default 
service name for tha kafka.