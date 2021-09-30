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
Timeseries DB for storing all the metrics
- Statefulset: This manifest includes the pod and volume 
configurations for Prometheus application
- Service: k8s service for prometheus. Grafana and Dashboard API 
uses that service to reach prometheus
- Configmap: This helm template basically creates a configmap to
mount prometheus pod. In addition to prometheus-config/rules.yaml and
prometheus-config/alerts.yaml we have the prometheus.yaml inside
that configmap which we configure scraping targets.
- ServiceAccount: Service account is used for allowing prometheus
to fetch metrics from k8s API

## Grafana
UI Component that fetchs and displays metrics from Prometheus
- Statefulset: This manifest includes the pod and volume 
configurations for Grafana application.
- Service: Grafana service is for opening up the service to 
the outside as a loadbalancer
- Secret: This file is for defining user credential for grafana.
You should put the credentials as base64 encoded like below.
```
admin-user: YWRtaW4=
admin-password: aG9tZWdyb3du
``` 
- Configmap: We're setting prometheus as a datasource with the
config files we defined inside that configmap.
- Dashboards-configmap: This template basically iterates over 
grafana-dashboard folder in this repo and print the json
content. As we are mounting this configmap to the pod, our 
default dashboards are being shown in the grafana automatically.

## Node Exporter
Linux server metrics exporter
- ds (Daemonset): Daemonset object type makes us enable to deploy
1 node exporter pod to every one of our nodes. We need that as 
we need to monitor all the nodes we have. Configurations are 
pretty similar with deployment/statefulset files. This daemonset
file deploys our node exporter pods.

## Kube State Metrics
Kube state metrics collects metrics for k8s objects like pods, volumes
- Deployment: Manifest for deploying kube state metrics.
- Service: For prometheus to reach and scrape kube state metrics.

## Kafka Exporter
Directly connects kafka and exports metrics from it

## JMX Exporter
Brokers/Zookeeper is the same jmx deployments but configured to
scrape different targets and metrics, one is for zookeeper,
one is for brokers.
- In the configmap files, the line 
```
hostPort: kafka-clusterip-jmx.default:39999
``` 
defines the jmx addresses for kafka, which we already opened 
for kafka brokers/zookeepers and create an additional jmx
service to point the jmx ports.
- JMX exporter basically communicates with JVM and fetch the 
metrics through that address.

## Event Exporter
Event Exporter fetchs the events on k8s, using k8s api.
- Deployment: deploys the event exporter using a public image
- Service: is for prometheus to scrape the events exporter
- ServiceAccount: Service Account is for granting permission 
to fetch events from kube api.

## Dashboard API
This is the api application we're using for collecting metrics
from prometheus and passing to the Dextrus dashboards. If you
haven't already, you need to build and push the prometheus api
project to your image registry, and pass your image address
to the values.yaml on helm chart.




 



