
Este repositório é referente ao meu curso de DevOps na Udemy:

http://dev-ops-ninja.com


# rancher-ha

Repositorio usado para mostrar instalação do Rancher em HA.

https://rancher.com/docs/rancher/v2.x/en/installation/how-ha-works/

## Requisitos

1 DNS
1 máquina para load balancer
3 máquinas para o rancher-server

Usando na demonstração: UBUNTU 16.04 LTS

## Docker instalado em todas as máquinas

```sh
$ sudo su
$ curl https://releases.rancher.com/install-docker/19.03.sh  | sh
$ usermod -aG docker ubuntu
```

## Portas

https://rancher.com/docs/rancher/v2.x/en/installation/requirements/ports/


## RKE

https://rancher.com/docs/rancher/v2.x/en/installation/k8s-install/create-nodes-lb/


Why three nodes?

In an RKE cluster, Rancher server data is stored on etcd. This etcd database runs on all three nodes.

The etcd database requires an odd number of nodes so that it can always elect a leader with a majority of the etcd cluster. If the etcd database cannot elect a leader, etcd can suffer from split brain, requiring the cluster to be restored from backup. If one of the three etcd nodes fails, the two remaining nodes can elect a leader because they have the majority of the total number of etcd nodes.


## INCIO

Logar na máquina do ELB - onde tudo será realizado
Instalar o kubectl nela também
Instalar o RKE nela também.

```sh
$ ssh -i devops-ninja.pem ubuntu@35.175.118.212   # - NGINX - LB

$ ssh -i devops-ninja.pem ubuntu@18.206.46.208   # - rancher-server-1
$ ssh -i devops-ninja.pem ubuntu@3.237.96.159    # - rancher-server-2
$ ssh -i devops-ninja.pem ubuntu@34.234.225.242   # - rancher-server-3

# Instalar Kubectl
$ curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
$ chmod +x ./kubectl
$ mv ./kubectl /usr/local/bin/kubectl
$ kubectl version --client


# Instalar RKE
$ curl -LO https://github.com/rancher/rke/releases/download/v1.1.7/rke_linux-amd64
$ mv rke_linux-amd64 rke
$ chmod +x rke
$ mv ./rke /usr/local/bin/rke
$ rke --version


$ exit


# COPIAR rancher-cluster.yml para Servidor

# USAR user ubuntu
# Copiar o PEM e colar no ARQUIVO.
$ vi ~/.ssh/id_rsa
$ chmod 600 /home/ubuntu/.ssh/id_rsa


# Rodar RKE
$ rke up --config ./rancher-cluster.yml



# Após o cluster subir:...
$ export KUBECONFIG=$(pwd)/kube_config_rancher-cluster.yml
$ kubectl get nodes
$ kubectl get pods --all-namespaces

# SALVAR OS ARQUIVOS



# Instalar HELM
$ curl -LO https://get.helm.sh/helm-v3.3.1-linux-amd64.tar.gz
$ tar -zxvf helm-v3.3.1-linux-amd64.tar.gz
$ sudo mv linux-amd64/helm /usr/local/bin/helm


# Instalar o Rancher - Preparar
$ helm repo add rancher-stable https://releases.rancher.com/server-charts/stable
$ kubectl create namespace cattle-system


# Certificate Manager
$ kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v0.15.0/cert-manager.crds.yaml
$ kubectl create namespace cert-manager
$ helm repo add jetstack https://charts.jetstack.io

$ helm repo update
$ helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --version v0.15.0

$ kubectl get pods --namespace cert-manager


# Instalar Rancher
$ helm install rancher rancher-stable/rancher \
  --namespace cattle-system \
  --set hostname=rancher.dev-ops-ninja.com


# Verificar deployment
$ kubectl -n cattle-system rollout status deploy/rancher
$ kubectl -n cattle-system get deploy rancher


# RODAR O NGINX
$ sudo vi /etc/nginx.conf
$ docker run -d --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  -v /etc/nginx.conf:/etc/nginx/nginx.conf \
  nginx:1.14
```


# Kubernetes-HA - Alta Disponibilidade


Repositorio usado para mostrar instalação do Rancher em HA.

https://rancher.com/docs/rancher/v2.x/en/troubleshooting/kubernetes-components/etcd/

https://rancher.com/learning-paths/building-a-highly-available-kubernetes-cluster/


## Requisitos

Cluster Kubernetes HA de Produção

3 instâncias para ETCD - Podendo perder 1
2 instâncias para CONTROLPLANE - Podendo perder 1
4 instâncias para WORKER - Podendo perder todas

Usando na demonstração: UBUNTU 16.04 LTS

## Docker instalado em todas as máquinas

```sh
$ sudo su
$ curl https://releases.rancher.com/install-docker/19.03.sh  | sh
$ usermod -aG docker ubuntu
```


## INCIO

Abrir o Rancher e criar um novo cluster.

Adicionar novo cluster com Existing Nodes


```sh
$ ssh -i devops-ninja.pem ubuntu@3.227.241.169   # - Rancher-server

#ETCD
$ ssh -i devops-ninja.pem ubuntu@34.200.230.114  # - etcd-1
$ ssh -i devops-ninja.pem ubuntu@3.238.62.131    # - etcd-2
$ ssh -i devops-ninja.pem ubuntu@3.230.119.189   # - etcd-3

#CONTROLPLANE
$ ssh -i devops-ninja.pem ubuntu@3.238.34.100  # - controlplane-1
$ ssh -i devops-ninja.pem ubuntu@3.236.176.198 # - controlplane-2

#WORKER
$ ssh -i devops-ninja.pem ubuntu@34.205.53.204 # - worker-1
$ ssh -i devops-ninja.pem ubuntu@3.236.174.43  # - worker-2
$ ssh -i devops-ninja.pem ubuntu@3.80.162.150  # - worker-3
$ ssh -i devops-ninja.pem ubuntu@3.237.75.239  # - worker-4


# docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run rancher/rancher-agent:v2.5.0 --server https://3.227.241.169 --token zw9dgzb99n7fkg7l7lsb4wn6p49gmhcfjdp9chpzllzgpnjg9gv967 --ca-checksum 7c481267daae071cd8ad8a9dd0f4c5261038889eccbd1a8e7b0aa1434053731b --node-name etcd-1 --etcd

# docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run rancher/rancher-agent:v2.5.0 --server https://3.227.241.169 --token zw9dgzb99n7fkg7l7lsb4wn6p49gmhcfjdp9chpzllzgpnjg9gv967 --ca-checksum 7c481267daae071cd8ad8a9dd0f4c5261038889eccbd1a8e7b0aa1434053731b --node-name etcd-2 --etcd

# docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run rancher/rancher-agent:v2.5.0 --server https://3.227.241.169 --token zw9dgzb99n7fkg7l7lsb4wn6p49gmhcfjdp9chpzllzgpnjg9gv967 --ca-checksum 7c481267daae071cd8ad8a9dd0f4c5261038889eccbd1a8e7b0aa1434053731b --node-name etcd-3 --etcd

# docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run rancher/rancher-agent:v2.5.0 --server https://3.227.241.169 --token zw9dgzb99n7fkg7l7lsb4wn6p49gmhcfjdp9chpzllzgpnjg9gv967 --ca-checksum 7c481267daae071cd8ad8a9dd0f4c5261038889eccbd1a8e7b0aa1434053731b --node-name controlplane-1 --controlplane

# docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run rancher/rancher-agent:v2.5.0 --server https://3.227.241.169 --token zw9dgzb99n7fkg7l7lsb4wn6p49gmhcfjdp9chpzllzgpnjg9gv967 --ca-checksum 7c481267daae071cd8ad8a9dd0f4c5261038889eccbd1a8e7b0aa1434053731b --node-name controlplane-2 --controlplane

# docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run rancher/rancher-agent:v2.5.0 --server https://3.227.241.169 --token zw9dgzb99n7fkg7l7lsb4wn6p49gmhcfjdp9chpzllzgpnjg9gv967 --ca-checksum 7c481267daae071cd8ad8a9dd0f4c5261038889eccbd1a8e7b0aa1434053731b --node-name worker-1 --worker

# docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run rancher/rancher-agent:v2.5.0 --server https://3.227.241.169 --token zw9dgzb99n7fkg7l7lsb4wn6p49gmhcfjdp9chpzllzgpnjg9gv967 --ca-checksum 7c481267daae071cd8ad8a9dd0f4c5261038889eccbd1a8e7b0aa1434053731b --node-name worker-2 --worker


```



