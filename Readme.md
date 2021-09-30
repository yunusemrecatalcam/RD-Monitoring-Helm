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

### Kafka exporter config
- On the kafka-exporter/kafka-exporter-deployment.yaml,
the line 
```
'--kafka.server=kafka:9092'
```
should point the kafka service, change it if necessary. 
Probably you won't need to change it's the default 
kafka service name on Dextrus Clusters.


# Components
We have all the components as a seperate folder placed in the
templates folder to keep the organized.

## Prometheus
- Statefulset: This manifest includes the pod and volume 
configurations for prometheus application
- Service: k8s service for prometheus. Grafana and Dashboard API 
uses that service to reach prometheus
- Configmap: This helm template basically creates a configmap to
mount prometheus pod. In addition to prometheus-config/rules.yaml and
prometheus-config/alerts.yaml we have the prometheus.yaml inside
that configmap which we configure scraping targets.
- ServiceAccount: Service account is used for allowing prometheus
to fetch metrics from k8s API





