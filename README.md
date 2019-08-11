# kubernetes-prometheus-monitoring
Monitoring kubernetes cluster with prometheus and graphana


This solution is enable monitoring in kubernetes cluster with graphana dashboard.

Prometheus is an open source monitoring framework. Explaining Prometheus is out of the scope of this article. In this article, I will guide you to setup Prometheus on a Kubernetes cluster and collect node, pods and services metrics automatically using Kubernetes service discovery configurations.


```	
kubectl create namespace monitoring 

```
You need to assign cluster reader permission to this namespace so that Prometheus can fetch the metrics from kubernetes API’s.

```
kubectl create -f prometheus-clusterrole.yaml

```
Create A Config Map
We should create a config map with all the prometheus scrape config and alerting rules, which will be mounted to the Prometheus container in /etc/prometheus as prometheus.yaml and prometheus.rules files. The prometheus.yaml contains all the configuration to dynamically discover pods and services running in the kubernetes cluster. prometheus.rules will contain all the alert rules for sending alerts to alert manager.

```
kubectl create -f prometheus-configmap.yaml -n monitoring

```

Create A Prometheus Deployment

```
kubectl create -f prometheus-deployment.yaml -n monitoring

```

Exposing Prometheus As A Service

```

kubectl create -f prometheus-service.yaml -n monitoring

```
Go to prometheus webpage to query for metrices.
http://<NodeIP>:<Nodeport>


Deploy kube state metris 

Wiki:- 
kube-state-metrics vs. metrics-server
The metrics-server is a project that has been inspired by Heapster and is implemented to serve the goals of core metrics pipelines in Kubernetes monitoring architecture. It is a cluster level component which periodically scrapes metrics from all Kubernetes nodes served by Kubelet through Summary API. The metrics are aggregated, stored in memory and served in Metrics API format. The metric-server stores the latest values only and is not responsible for forwarding metrics to third-party destinations.

kube-state-metrics is focused on generating completely new metrics from Kubernetes' object state (e.g. metrics based on deployments, replica sets, etc.). It holds an entire snapshot of Kubernetes state in memory and continuously generates new metrics based off of it. And just like the metric-server it too is not responsibile for exporting its metrics anywhere.

Having kube-state-metrics as a separate project also enables access to these metrics from monitoring systems such as Prometheus.
https://github.com/kubernetes/kube-state-metrics 

```
kubectl create -f kube-state-*.yaml 

```

Deploy Node exporter. This is prometheus agent runs as a daemonset ( on each Node) to get metrices.

```
kubectl create -f node-exporter-daemonset.yaml -n monitoring

```

Deploy Graphana

```
kubectl create -f graphana-deployment.yaml

```
Expose graphana as a service of your choice ( NodePort or LoadBalancer). In our case i decided to go with Nodeport. 

```
kubectl expose deployment grafana --type=NodePort --port=3000 --target-port=3000 --namespace=monitoring
```
Note:- You need to add the generated nodeport in the firewall. In my case i have added the port as an incoming rule in Security group in AWS.
```
kubectl get svc <graphan service name> -n monitoring ( You can get the nodeport)
```

Now, configure your cluster setting on Grafana.
1. Enable the app at http://{Nodeip}:{NodePort}/plugins/grafana-kubernetes-app/edit
2  Add a data source of prometheus type: http://{Nodeip}:{NodePort}/datasources/new

3. Add a cluster at http://{IP}:3000/plugins/grafana-kubernetes-app/page/cluster-config with username/password and CA certificate.
Select the datasource which you create in step 2 and do validate.

you can get the cluster ip and user/password from ~/.kube/config of the master server. ( Hint)

Yeah! Now you finally get the three built-in dashboards working. :-)


