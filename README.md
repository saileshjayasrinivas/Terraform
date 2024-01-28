
Logging Setup with Grafana, Prometheus, and Loki on AWS

This repository contains configurations and instructions for setting up logging infrastructure using Grafana, Prometheus, and Loki on AWS.

Prometheus Setup
Add Helm repository and update:

helm repo add Prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
Update Prometheus.yaml file as needed. Modify configurations such as jobs, retention, and volume size.

yaml
# Example configuration for jobs and retention
scrape_configs:
  - job_name: Port_PlutoMetrics
    scheme: http
    static_configs:
      - targets: ['IP']
    basic_auth:
      username: "admin"
      password: "password"

retention: "90d"

persistentVolume:
  size: <size_as_required>
Install Prometheus using Helm:

helm install prometheus Prometheus-community/prometheus --namespace prometheus --values prometheus.yaml

Grafana Setup
Create Grafana namespace:
kubectl create namespace grafana
Add Grafana Helm repository:
helm repo add grafana https://grafana.github.io/helm-charts
Install Grafana using Helm:

helm install grafana grafana/grafana \
  --namespace grafana \
  --set persistence.storageClassName="gp2" \
  --set persistence.enabled=true \
  --set adminPassword='password' \
  --set datasources."datasources\.yaml".apiVersion=1 \
  --set datasources."datasources\.yaml".datasources[0].name=Prometheus \
  --set datasources."datasources\.yaml".datasources[0].type=prometheus \
  --set datasources."datasources\.yaml".datasources[0].url=http://prometheus-server.prometheus.svc.cluster.local \
  --set datasources."datasources\.yaml".datasources[0].access=proxy \
  --set datasources."datasources\.yaml".datasources[0].isDefault=true \
  --set service.type=LoadBalancer
  
Loki Setup
Create logging namespace:

kubectl create ns logging
Add Loki Helm repository:

helm repo add grafana https://grafana.github.io/helm-charts
Update loki.yaml file with necessary changes, including bucket name, access key, secret key, and node selector.

# Example configuration for Loki
loki:
  bucketname: <your_bucket_name>
  accesskey: <your_access_key>
  secretkey: <your_secret_key>
  nodeselector: <your_node_selector>

Update s3 values as well 
#Repace values as per your Env  
s3://accesskey:secret-key@ap-south-1/s3-bucketname 


Install Loki using Helm:

helm upgrade --install loki grafana/loki-simple-scalable -n logging -f loki.yaml
Install Promtail:

Update the lokiAddress in promtail
ex:- http://loki-write.logging.svc.cluster.local:3100/loki/api/v1/push 

helm upgrade --install promtail grafana/promtail -f promtail.yaml -n logging
Apply Loki Ingress:

kubectl apply -f loki_ingress.yaml

Once the setup is complete, add the Loki datasource in Grafana using the provided URL.

Example URL: http://loki-read.eu-logging.svc.cluster.local:3100

To access Grafana globally, obtain the LoadBalancer URL, and either associate it with a DNS entry or configure an Ingress that is accessible under whitelisted or private networks.










