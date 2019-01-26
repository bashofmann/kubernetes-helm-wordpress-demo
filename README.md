# Deploying ingress controller, cert-manager, external-dns, percona-operator, percona-cluster, wordpress

## Install an nginx ingress controller to route traffic to ingress resources over a single LBaaS external IP

helm upgrade --install nginx-ingress stable/nginx-ingress -f ingress-controller.yaml --namespace kube-system

## Install cert-manager and a cluster issuer to automatically retrieve LetsEncrypt certificates for an ingress resource

helm upgrade --install cert-managerstable/cert-manager --namespace kube-system 
kubectl apply -f cluster-issuer.yaml --namespace kube-system

## Install external-dns to automatically create DNS entries for ingress resources

kubectl apply -f external-dns.yaml --namespace kube-system

## Install a percona mysql operator that allows to easily create highly availabe percona clusters by adding a new mysql cluster resource

helm repo add presslabs https://presslabs.github.io/charts
helm upgrade --install mysql-operator presslabs/mysql-operator --namespace kube-system

## Create a percona mysql cluster

kubectl create namespace wordpress
kubectl apply -f mysql-cluster.yaml --namespace wordpress

## Forward the port of the mysql cluster to localhost

kubectl --namespace wordpress port-forward service/mysql-cluster-mysql 3306:3306

## Connect to the mysql cluster via localhost

mysql -h 127.0.0.1 -P 3306 -u root -p

## Create a database for wordpress

create database bitnami_wordpress;

## Install the wordpress chart using the percona mysql database and creating an ingress resource with a TLS certificate and external-dns

helm upgrade --install my-wordpress stable/wordpress --namespace wordpress -f wordpress.yaml

## Connect to the Percona MySql orchestrator over localhost

kubectl port-forward mysql-operator-orchestrator-0 3000 -n kube-system

# LinkerD

## Installation in cluster

curl -sL https://run.linkerd.io/install | sh

export PATH=$PATH:$HOME/.linkerd2/bin

linkerd version

linkerd check --pre

linkerd install | kubectl apply -f -

## Inject LinkerD proxies into existing wordpress deployment

kubectl get deployment my-wordpress-wordpress -o yaml | linkerd inject - | kubectl apply -f -

# Kubeless

## Install Kubeless

export RELEASE=$(curl -s https://api.github.com/repos/kubeless/kubeless/releases/latest | grep tag_name | cut -d '"' -f 4)
kubectl create ns kubeless
kubectl create -f https://github.com/kubeless/kubeless/releases/download/$RELEASE/kubeless-$RELEASE.yaml

## Deploy and call function into Kubeless

kubeless function deploy uppercase --runtime nodejs8 --from-file uppercase.js --handler handler.uppercase --namespace kubeless

kubeless function ls uppercase

kubeless function call uppercase --data 'Hello world!'

# Auto-Scaling

https://docs.syseleven.de/metakube/de/tutorials/use-horizontal-pod-autoscaling
https://docs.syseleven.de/metakube/de/tutorials/use-horizontal-node-autoscaling

# Backups

https://docs.syseleven.de/metakube/de/tutorials/create-backup-and-restore

# Symfony demo project with Dockerfile and kubernetes resources

https://github.com/bashofmann/symfony-demo-kubernetes

# Development flow project mit Forge and Telepresence

https://github.com/bashofmann/kubernetes-dev-flow-demo

## Setup forge

forge setup

## Deploy to kubernetes with forge

forge deploy
forge --profile stage deploy

## Swap existing pod with locally running process

telepresence --swap-deployment quote-svc --namespace dev-flow-demo --expose 3000 --run npm run debug

# Cluster Overview Web UI

https://www.weave.works/oss/scope/

Helm chart: https://github.com/helm/charts/tree/master/stable/weave-scope
