<h2>Ingress Controller </h2>

Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. Traffic routing is controlled by rules defined on the Ingress resource.

<h2>Nginx Ingress Controller </h2>

It is built around the Kubernetes Ingress resource, using a ConfigMap to store the NGINX configuration. We Will be using helm to Setup nginx ingress controller.

```
helm install \
stable/nginx-ingress \
--name my-nginx \
--set rbac.create=true

helm status my-nginx 
```

<h2>Jenkins Setup using helm with Ingress Enabled. </h2>

Edit the Ingress configuration in values.yaml. Endpoint "jenkins-one.debian.com"

```
ingress:
    enabled: true
    apiVersion: "extensions/v1beta1"
    labels: {}
    annotations:
     kubernetes.io/ingress.class: nginx
    hostName: jenkins-one.debian.com
    tls:

```

Helm to install the jenkins 

```
helm install --name cicd stable/jenkins --set rbac.create=true -f values.yaml

helm status cicd
``````
Use endpoint "jenkins-one.debian.com" to access the Jenkinsand password for admin user can fetch from below command.

```
1. Get your 'admin' user password by running:
  printf $(kubectl get secret --namespace default cicd-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo

2. Visit http://jenkins-one.debian.com
```

<h2>Setting Monitoring Stack using helm </h2>

Create the separate namesapce

```
kubectl create namespace monitoring 
kubectl get ns 
```

Edit the Values.yaml and Add Ingress configurtions. Will be using persistentVolume for both prometheus and alertmanger. 

```
helm install --name prometheus stable/prometheus \
    --namespace prometheus \
    --set alertmanager.persistentVolume.storageClass="gp2" \
    --set server.persistentVolume.storageClass="gp2" -f values.yaml 
    
helm status prometheus
```

Prometheus and Alertmanager can access using below endpoint.
    
```
The Prometheus server can be accessed via port 80 on the following DNS name from within your cluster:
prometheus-server.prometheus.svc.cluster.local

From outside the cluster, the server URL(s) are:
http://prometheus-one.debian.com

From outside the cluster, the alertmanager URL(s) are:
http://alertmanager-one.debian.com

```


