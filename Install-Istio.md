# Chapter 09 - Installing Istio

The commands given in the Chapter 09 are given here for copy and paste if you are reading the printed book.

## Make sure that DNS is working
```
$ dig +search +noall +answer ibm.com
```

## Check Istio releases
```
$ curl -L -s https://api.github.com/repos/istio/istio/releases |
grep tag_name
```

## Find out latest version
```
$ export ISTIO_VERSION=$(curl -L -s
https://api.github.com/repos/istio/istio/releases/latest | grep
tag_name | sed "s/ *\"tag_name\": *\"\\(.*\\)\",*/\\1/")

$ echo $ISTIO_VERSION
```

## We will work with Istio 1.3.5

```
$ cd ## Switch to your home directory
$ export ISTIO_VERSION=1.3.5
$ curl -L https://git.io/getLatestIstio | sh -
$ cd istio-$ISTIO_VERSION
```

## Edit .bashrc file

```
$ vi ~/.bashrc

export ISTIO_VERSION=1.3.5
if [ -d ~/istio-${ISTIO_VERSION}/bin ] ; then
export PATH="~/istio-${ISTIO_VERSION}/bin:$PATH"
fi
```

## Source .bashrc file

```
$ source ~/.bashrc
```

## Verify Istio Install

```
$ istioctl verify-install
```

## Reset Helm

```
$ helm reset --force
```

## Create istio-system namespace

```
$ kubectl create namespace istio-system
```

## Grant cluster admin

```
$ kubectl create clusterrolebinding istio-system-cluster-rolebinding
--clusterrole=cluster-admin --serviceaccount=istiosystem:
default
```

## Install Istio CRDs

```
$ cd ~/istio-$ISTIO_VERSION/install/kubernetes/helm/istioinit/
files
$ for i in ./crd*yaml; do kubectl apply -f $i; done
```

## Create tiller service account

```
$ cd ~/istio-$ISTIO_VERSION/install/kubernetes/helm
$ kubectl apply -f helm-service-account.yaml
$ helm init --service-account tiller
```

## Validate tiller pod

```
$ kubectl get pods -n kube-system | grep tiller
```

## Generate Istio yaml

```
$ cd ~/istio-$ISTIO_VERSION
$ helm template install/kubernetes/helm/istio --name istio \
--namespace istio-system \
--values install/kubernetes/helm/istio/values-istio-demo.yaml | \
kubectl apply -f -
```

## Apply generated Istio yaml

```
$ cd ~/istio-$ISTIO_VERSION
$ helm template install/kubernetes/helm/istio --name istio \
--namespace istio-system \
--values install/kubernetes/helm/istio/values-istio-demo.yaml | \
kubectl apply -f -
```

## Watch pods (Press ctrl-c when all are ready)

```
$ kubectl -n istio-system get pods --watch
```

## Install Istio using Helm and Tiller

### Delete existing Istio
```
$ cd ~/istio-$ISTIO_VERSION
$ helm template install/kubernetes/helm/istio --name istio \
--namespace istio-system \
--values install/kubernetes/helm/istio/values-istio-demo.yaml |\
kubectl delete -f -
```

### Create Istio CRD 
```
$ cd ~/istio-$ISTIO_VERSION/install/kubernetes/helm
$ helm install ./istio-init --name istio-init --namespace istiosystem
```

### Install Istio
```
$ helm install ./istio -f istio/values-istio-demo.yaml \
--name istio --namespace istio-system
```

### Check deployment
```
$ kubectl -n istio-system get deployment
```

## Installing Istio using a demo profile

### Delete earlier Helm 

```
$ helm del --purge istio

$ helm del --purge istio-init
```

### Install Istio using a demo profile

```
$ cd ~/istio-$ISTIO_VERSION/
$ kubectl apply -f install/kubernetes/istio-demo.yaml
```

## Verify Install


### Check version
```
$ istioctl version --short
```

### Check pods
```
$ kubectl -n istio-system get pods
```

## Installing a load balancer

### Make sure that ip_vs is installed and label the node
```
$ sudo lsmod | grep ^ip_vs

$ sudo ipvsadm -ln

$ echo "ip_vs" | sudo tee /etc/modules-load.d/ipvs.conf

$ kubectl label node osc01.servicemesh.local proxy=true

$ helm repo add kaal https://servicemeshbook.github.io/keepalived

$ helm repo update
```

### Update Helm repo, grant cluster admin and install the load balancer

```
$ helm repo update

$ kubectl create clusterrolebinding \
keepalived-cluster-role-binding \
--clusterrole=cluster-admin --serviceaccount=keepalived:default
```

### Install load balancer
```
$ helm install kaal/keepalived --name keepalived \
--namespace keepalived \
--set keepalivedCloudProvider.serviceIPRange="192.168.142.248/29" \
--set nameOverride="lb"
```

### Check pods

```
$ kubectl -n keepalived get pods
```

### Check services

```
$ kubectl -n istio-system get services
```

## Enabling Istio

### For an existing application

```
$ cd ~/servicemesh
$ istioctl kube-inject -f bookinfo.yaml > bookinfo_proxy.yaml
$ cat bookinfo_proxy.yaml
```

### Check the differences in yaml generated

```
$ diff -y bookinfo.yaml bookinfo_proxy.yaml
```

### Deploy bookinfo

```
$ kubectl -n istio-lab apply -f bookinfo_proxy.yaml
```

### Check pods

```
$ kubectl -n istio-lab get pods
```

### For a new application


#### Delete, create and label the namespace

```
$ kubectl delete namespace istio-lab

$ kubectl create namespace istio-lab

$ kubectl label namespace istio-lab istio-injection=enabled
```

### Deploy application
```
$ kubectl -n istio-lab apply -f ~/servicemesh/bookinfo.yaml
```

## Horizontal Pod Scaling

```
$ kubectl -n istio-system get hpa
```

### How to delete HPA

```
$ kubectl -n istio-system delete hpa --all

$ kubectl -n istio-system scale deploy istio-egressgateway --replicas=1
$ kubectl -n istio-system scale deploy istio-ingressgateway --replicas=1
$ kubectl -n istio-system scale deploy istio-pilot --replicas=1
$ kubectl -n istio-system scale deploy istio-policy --replicas=1
$ kubectl -n istio-system scale deploy istio-telemetry --replicas=1
```







