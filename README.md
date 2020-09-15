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
$ ssh -i curso.pem ubuntu@34.225.194.69   # - NGINX - LB

$ ssh -i curso.pem ubuntu@3.229.130.187   # - rancher-server-1
$ ssh -i curso.pem ubuntu@3.236.87.190    # - rancher-server-2
$ ssh -i curso.pem ubuntu@3.227.234.206   # - rancher-server-3

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
  --set hostname=rancher.sagemaker.io


# Verificar deployment
$ kubectl -n cattle-system rollout status deploy/rancher
$ kubectl -n cattle-system get deploy rancher


# RODAR O NGINX
$ docker run -d --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  -v /etc/nginx.conf:/etc/nginx/nginx.conf \
  nginx:1.14
```





