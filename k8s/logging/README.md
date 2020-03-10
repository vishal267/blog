<h2>Logging for kubernetes cluster </h2>


We will be deploying EFK stack for Logging in kubernetes cluster. 

Elasticsearch - Its used to store logs in form of indexes. 

Fluent-bit is an open source Log processor and forwarder. Its used for collecting logs from differenct sources, unify and semd them to multiple destinations. We will be running daemonsets of fluent-bit 

kibana - It's provide interface for searching and visualizing logs.


ElasticSearch Cluster - Running ES Cluster in Kubernetes. 

```
kubectl create ns logging 
kubectl get ns 

helm install --name elasticsearch stable/elasticsearch \
    --set master.persistence.enabled=false \
    --set data.persistence.enabled=false \
    --namespace logging 
    
```
       
