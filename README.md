# Canary-Deployment-Using-Istio-on-EKS-Cluster-Istio-Kiali-Grafana

<p align="center">
  <img width="1000" height="600" src="https://miro.medium.com/max/2560/1*rvg_zBBHwD6kJshpW9vmaw.gif">
</p> 

## Take away from this article:
1. Creating aws profile.
2. Creating eks cluster through eksctl.
3. Installing istio inside eks cluster.
4. Launching sample book application using istio.
5. Visualizing traffic of application using kiali.
6. Routing the traffic to specific version of application using virtual service.
7. Visualizing traffic and cluster configuration using grafana.
8. Canary deployment using Istio.

Istio is an open-source service mesh implementation that manages communication and data sharing between microservices. The platform is added to reduce the complexity of managing network services.
Once installed, it injects proxies inside a Kubernetes pod, next to the application container. Each proxy is configured to intercept requests and route traffic to the appropriate service while applying policies.

## What is Service Mesh ?

<p align="center">
  <img width="1000" height="600" src="https://miro.medium.com/max/918/0*-QfeoUAXaPuP7I1w">
</p> 

A service mesh is a way to control how different parts of an application share data with one another. Unlike other systems for managing this communication, a service mesh is a dedicated infrastructure layer built right into an app. This visible infrastructure layer can document how well (or not) different parts of an app interact, so it becomes easier to optimize communication and avoid downtime as an app grows.

Each part of an app, called a “service,” relies on other services to give users what they want. If a user of an online retail app wants to buy something, they need to know if the item is in stock. So, the service that communicates with the company’s inventory database needs to communicate with the product webpage, which itself needs to communicate with the user’s online shopping cart. To add business value, this retailer might eventually build a service that gives users in-app product recommendations. This new service will communicate with a database of product tags to make recommendations, but it also needs to communicate with the same inventory database that the product page needed — it’s a lot of reusable, moving parts.

Modern applications are often broken down in this way, as a network of services each performing a specific business function. In order to execute its function, one service might need to request data from several other services. But what if some services get overloaded with requests, like the retailer’s inventory database? This is where a service mesh comes in — it routes requests from one service to the next, optimizing how all the moving parts work together.

## Why Istio ?

<p align="center">
  <img width="1000" height="600" src="https://miro.medium.com/max/3840/0*ytaZ0lUtw5LKNFRE.jpg">
</p> 


Istio enables organizations to secure, connect, and monitor microservices, so they can modernize their enterprise apps more swiftly and securely. Istio manages traffic flows between services, enforces access policies, and aggregates telemetry data, all without requiring changes to application code.

## Installing AWS Cli:

```
# downloading aws cli software
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" 

# Unzip aws cli
unzip awscliv2.zip 

# install aws cli
sudo ./aws/install
```

<p align="center">
  <img width="1000" height="105" src="https://miro.medium.com/max/1050/1*cU1wlY9Q1DwDj58eABEaJA.png">
</p> 

<p align="center">
  <img width="1000" height="50" src="https://miro.medium.com/max/1050/1*JTCjuZNMnffYN-uOi777Ig.png">
</p> 

<p align="center">
  <img width="1000" height="110" src="https://miro.medium.com/max/1050/1*dqGXC4mRCWGazt6IqjgY1w.png">
</p> 

## Creating Profile in aws cli:
We have to create a profile that will take access key and secret access key to authenticate the user to provision the eks cluster on aws.

```
aws configure --profile  istio

aws configure list-profiles
```

<p align="center">
  <img width="1000" height="150" src="https://miro.medium.com/max/1050/1*wrBKHLv71i9WUmSREqcHMA.png">
</p> 

As you can see that i have created istio profile. you can create using any name. It will ask you for aws access key, aws secret access key and region. Enter them respectively and in which region your are working enter that region name.

## Install kubectl:
Kubectl is the command line configuration tool for Kubernetes that communicates with a Kubernetes API server. Using kubectl allows you to create, inspect, update, and delete Kubernetes objects. Below is the command that will help you to create a kubernetes repo and install kubectl on your local.

```
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

sudo yum install -y kubectl
```

<p align="center">
  <img width="1000" height="400" src="https://miro.medium.com/max/1050/1*b1seEh-Repip2maLEStQig.png">
</p> 

<p align="center">
  <img width="1000" height="100" src="https://miro.medium.com/max/1050/1*RRwDF27E5P379hS52fBZuQ.png">
</p> 

## Installing eksctl:
eksctl is a simple CLI tool for creating and managing clusters on EKS - Amazon's managed Kubernetes service for EC2.

```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

<p align="center">
  <img width="1000" height="230" src="https://miro.medium.com/max/1050/1*_zd_BKmY1FXSWQ5U8AS15A.png">
</p> 

## Creating EKS Cluster using eksctl:
We will be creating EKS cluster on aws using eksctl. First we will be creating a profile in aws configure. eksctl is a simple CLI tool for creating and managing clusters on EKS - Amazon's managed Kubernetes service for EC2.

<p align="center">
  <img width="1000" height="230" src="https://miro.medium.com/max/1050/1*LSAKP3l1O-dxnpQE0ng9_A.png">
</p> 

Below command will create eks cluster with 2 nodes. The node type is m5.large launching in eu-west-1 region. I am using the aws credentials by istio profile which i have created above.
`eksctl create cluster --name mycluster123 --node 2 --node-type m5.large --managed --region eu-west-1 --profile istio`

<p align="center">
  <img width="1000" height="400" src="https://miro.medium.com/max/1050/1*Duy1Vb3XAiUUChMTxhqgcg.png">
</p> 

<p align="center">
  <img width="1000" height="200" src="https://miro.medium.com/max/1050/1*shz15SE73JSn94r9PnkDKA.png">
</p> 

As you can see that the eks cluster is up and running. So we will check the pods and node status using below commands.
Checking Nodes and pods in all name spaces:
The pods in all namespaces is running and the status of the nodes is ready. you can check using these commands.

```
# for checking nodes
kubectl get nodes -o wide

# for checking pods in all namespaces.
kubectl get pods --all-namespaces
```


