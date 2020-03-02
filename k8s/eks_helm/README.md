<h2>Setting EKS STACK </h2>

EC2 instance with Access key and secret key in environment variable 

Installing eksctl binary.


```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/download/latest_release/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
eksctl version
sudo mv -v /tmp/eksctl /usr/local/bin
```
       
Create Cluster.yaml file - Instance type is t3.small and region is eu-north1

```
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: k8s-cluster
  region: eu-north-1

nodeGroups:
  - name: ng-1
    instanceType: t3.small
    desiredCapacity: 3
```

Create the cluster using eksctl command 

```
eksctl  create cluster -f eks/cluster.yaml
```

Checking Cluster Health and Info 

```
Kubectl cluster-info
kubectl get nodes 

```


2 - ALB Ingress Controller 

Subnet Tags - 

     kubernetes.io/role/internal-elb must be set to 1 or `` for internal LoadBalancers.
    kubernetes.io/role/elb must be set to 1 or `` for internet-facing LoadBalancers.

Create IAM policy - ingressController-iam-policy

use this 

https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.0.0/docs/examples/iam-policy.json

Attach this policy to node/worker role.

Deploy RBAC Roles and RoleBindings

```
 kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.0.0/docs/examples/rbac-role.yaml
```

deploy the AWS ALB Ingress controller
```
curl -sS "https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.0.0/docs/examples/alb-ingress-controller.yaml" > alb-ingress-controller.yaml

Edit the AWS ALB Ingress controller YAML to include the clusterName of the Kubernetes

kubectl apply -f alb-ingress-controller.yaml
kubectl get pods -n kube-system

kubectl logs -n kube-system $(kubectl get po -n kube-system | egrep -o alb-ingress[a-zA-Z0-9-]+)
```


SSL For ALB 

```
# Generate self-signed certificate for a domain
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=one.debian.com"

# Add our new certificate to the AWS Certificate Manager
aws acm import-certificate --certificate file://tls.crt --private-key file://tls.key --region us-north1

```

Deployment and Ingress configuration for Demo App
```
APP Deployment  file
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blog
spec:
  selector:
    matchLabels:
      app: blog
  replicas: 3
  template:
    metadata:
      labels:
        app: blog
    spec:
      containers:
      - name: blog
        image: dockersamples/static-site
        env:
        - name: AUTHOR
          value: blog
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: blog
  name: blog
spec:
  type: NodePort
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: blog

Ingress file

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: blog
  labels:
    app: blog
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS":443}]'
    alb.ingress.kubernetes.io/certificate-arn: "arn:aws:acm:eu-north-1:164366481040:certificate/xxxxxxxxx"
spec:
  rules:
   - host: one.debian.com
     http:
        paths:
          - path: /*
            backend:
              serviceName: blog
              servicePort: 80
```


Helm

Helm is a package manager for Kubernetes that allows developers and operators to more easily package, configure, and deploy applications and services onto Kubernetes clusters.



Installation 

```
#Helm Installation -

helm init

kubectl create serviceaccount --namespace kube-system tiller
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'
```


Monitoring Stack 

Deploy Prometheus using Persistent Storage

```
kubectl create namespace prometheus

helm install --name prometheus stable/prometheus \
    --namespace prometheus \
    --set alertmanager.persistentVolume.storageClass="gp2" \
    --set server.persistentVolume.storageClass="gp2"

kubectl get all -n prometheus

kubectl port-forward -n prometheus deploy/prometheus-server 8080:9090

```

Deploy Grafana using Persistent Storage

```
kubectl create namespace grafana
helm install grafana stable/grafana \
    --namespace grafana \
    --set persistence.storageClassName="gp2" \
    --set adminPassword='EKS!sAWSome' \
    --set datasources."datasources\.yaml".apiVersion=1 \
    --set datasources."datasources\.yaml".datasources[0].name=Prometheus \
    --set datasources."datasources\.yaml".datasources[0].type=prometheus \
    --set datasources."datasources\.yaml".datasources[0].url=http://prometheus-server.prometheus.svc.cluster.local \
    --set datasources."datasources\.yaml".datasources[0].access=proxy \
    --set datasources."datasources\.yaml".datasources[0].isDefault=true \
    --set service.type=LoadBalancer

kubectl get all -n grafana

username admin

kubectl get secret --namespace grafana grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

Jenkins for CI

```
helm install --name cicd stable/jenkins --set rbac.create=true,master.servicePort=80,master.serviceType=LoadBalancer

kubectl get pods -w

For Admin password 

printf $(kubectl get secret --namespace default cicd-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo

```
