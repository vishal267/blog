Kubernetes (k8s) is an open-source system for automating deployment, scaling, and management of containerized applications.
We can schedule containers on a cluster of servers.
We can run multiple containers on one server.
K8s manage the state of theses containers
Advantages of Kubernetes
We can run kubernetes on premise(Own datacenter) , public cloud (AWS , Google Cloud) and Hybrid cloud.
It a open source software which have huge community base.
Its modular which can be customised as per need.
Architecture Overview

 
Control Plane Components
kube-apiserver
etcd
kube-controller
kube-scheduler
Master Node
Responsible for the management of k8s cluster. Perform all administrative tasks and managing the worker nodes. Below are components of Master Node.
Kube-apiserver
Entry point for all REST commands used to control the cluster. It processes the REST requests, validates them and execute the same.
etcd storage
etcd storage acts as datastore which provide high available key value store for persisting cluster state. it stores object and config information.
Kube-controller-manager
Monitor the cluster state via the apiserver which check the shared state of the cluster and make changes to the current state to change it to desired one.
Example - Replication controller which take care of the number of pods in the system. it will automatically maintain the desired state of pod defined in yaml file.
Kube-Scheduler
It is responsible for deployment of configured pods and services onto nodes. it contain the information regarding resources available on the members of cluster, as well as the ones required for the configured.
Node Components
kubelet
kube-proxy
pod
Container Runtime Engine i.e Docker
Worker Node
The pods which contain the application are executed on worker node.It contain all services to manage the networking between the container, communicate with the master node and assign resources to the container scheduled. Below are components of worker node.
Kubelet
Kubelet gets the configuration of a pod from the apiserver and ensures that all defined containers are up and running. This is the worker service that's responsible for communicating with the master node. It also co-ordinate with etcd storage service running on master node to get information about services.
Kube-proxy
Kube-proxy acts as network proxy and a load balancer for a service on single worker node. Its also responsible for network routing for TCP and UDP packets.
Pods
Smallest unit of work of K8s , pods contains one or more containers that share volume, network and namespace.Pods are deployed on worker/minion node. Pods describe an application running on K8s. Pods are created, destroyed and recreated on demand based on the state of server and service
Container Runtime Engine
Docker is a container engine which runs on each of the worker nodes, and runs the configured pods. Its responsible for downloading the images and running the containers.
Minikube - For Local setup
Minikube is a tool that allow to run kubernetes cluster locally on single node.
It run on single machine inside a linux virtual machine.
Its used for testing and development purpose.
It works on windows, Linux and MacOS.
It required a virtualization software installed to run minikube.
Install Virtual Box and minikube
apt-get install virtualbox -y
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && sudo install minikube-linux-amd64 /usr/local/bin/minikube
Start K8s cluster using minikube

Install kubectl binary
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl
chmod +x kubectl && sudo mv kubectl /usr/local/bin/kubectl
Check the kubernetes cluster info:



Run a Echo server deployment with one pod. Below command will download a image which will run a container using this image.

Expose the Node port to host machine and check the minikube url using browser.

Open the URL "http://192.168.99.100:32351/"

Stop the kubernetes cluster
root@win:~# minikube stop
Stopping local Kubernetes cluster...
Machine stopped.

