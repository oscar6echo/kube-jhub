# JupyterHub Kube setup

Ref. [Zero to JupyterHub with Kubernetes](https://zero-to-jupyterhub.readthedocs.io/en/latest/).

## Config

```bash
export CLUSTERNAME=jhub
export EMAIL=olivier.borderies@gmail.com
export RELEASE=jhub
export NAMESPACE=jhub

```

## Setup Kube on GKE

```bash
##########################################
## auth
gcloud auth login
gcloud auth list

##########################################
## create cluster
gcloud container clusters create \
  --machine-type n1-standard-2 \
  --num-nodes 2 \
  --zone europe-west1-c \
  --cluster-version latest \
  $CLUSTERNAME

##########################################
## attach kubectl
gcloud container clusters get-credentials $CLUSTERNAME --zone europe-west1-c --project remote-dev-o6e

k version
k get-contexts
cat $HOME/.kube/config

k get node

##########################################
## create clusterrolebinding
kubectl create clusterrolebinding cluster-admin-binding \
  --clusterrole=cluster-admin \
  --user=$EMAIL

k get clusterrolebindings.rbac.authorization.k8s.io | grep cluster-admin-binding

# if error
# kubectl delete clusterrolebinding cluster-admin-binding

```

## Setup JupyterHub

```bash
##########################################
## init config
openssl rand -hex 32
nano config.yml


##########################################
## init helm
helm repo add jupyterhub https://jupyterhub.github.io/helm-chart/
helm repo list
helm repo update

##########################################
## install
helm upgrade --cleanup-on-fail \
  --install $RELEASE jupyterhub/jupyterhub \
  --namespace $NAMESPACE \
  --create-namespace \
  --version=0.9.0 \
  --values config.yml

kubectl --namespace=jhub get pod

# if error
# helm delete $RELEASE
# k delete namespace $NAMESPACE
# helm install again


##########################################
## helm paths https://helm.sh/docs/helm/helm/
# cache
ll -al $HOME/.cache/helm
ll -al $HOME/.cache/helm/repository
# config
ll -al $HOME/.config/helm
# data
ll -al $HOME/.local/share/helm


##########################################
## helm check what was deployed
mkdir -p helm
cp $HOME/.cache/helm/repository/jupyterhub-0.9.0.tgz helm/jupyterhub-0.9.0.tgz
tar zxvf helm/jupyterhub-0.9.0.tgz -C helm
helm template helm/jupyterhub --values config.yml --debug | tee helm/jhub-kube.yml > /dev/null
cat helm/jhub-kube.yml

##########################################
## set kubectl context
kubectl config set-context $(kubectl config current-context) --namespace ${NAMESPACE:-jhub}
k get pod

##########################################
## services
k get svc
k get svc proxy-public

# reaad external IP eg. 34.77.147.160


##########################################
## update config.yml
helm upgrade --cleanup-on-fail \
    $RELEASE jupyterhub/jupyterhub \
  --version=0.9.1 \
  --values config.yml

k get pod


```
