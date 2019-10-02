# Istio

## Prerquisites

Follow instructions to build your own Kubernetes environment by going through [https://github.com/servicemeshbook/byok](https://github.com/servicemeshbook/byok).

## Follow Chapter 8 - Install Demo Application
## Follow Chapter 9 - Install Istio

### Find out all previous releases of Istio
```
curl -L -s https://api.github.com/repos/istio/istio/releases | grep tag_name
```

### find out the latest version

```
export ISTIO_VERSION=$(curl -L -s https://api.github.com/repos/istio/istio/releases/latest | grep tag_name | sed "s/ *\"tag_name\": *\"\\(.*\\)\",*/\\1/")
echo $ISTIO_VERSION
```

### Download version 1.3.1 to stay consistent with the hands-on exercises for this book

```
cd ## Switch to your home directory
export ISTIO_VERSION=1.3.1
curl -L https://git.io/getLatestIstio | sh -
cd istio-$ISTIO_VERSION
```

### Edit your profile ~/.bashrc to include the following lines to add istioctl on system path

```
vi ~/.bashrc

export ISTIO_VERSION=1.3.1
if [ -d ~/istio-${ISTIO_VERSION}/bin ] ; then
    export PATH="~/istio-${ISTIO_VERSION}/bin:$PATH"
fi
```

### Source .bashrc to make changes to the system path

```
source ~/.bashrc
```

### Check if the current Kubernetes environment is ready for the install of chosen Istio version

```
istioctl verify-install
```

### Remove the tiller pod as we will initialize helm again

```
helm reset --force
```

### Create istio-system namespace which will be used by Istio

```
kubectl create namespace istio-system
```

### Grant cluster-admin role to the default service account for the namespace istio-system

```
kubectl create clusterrolebinding istio-system-cluster-role-binding --clusterrole=cluster-admin --serviceaccount=istio-system:default
```

### Install the Istio CRDs and re-initialize tiller to include istio-system namespace

```
cd ~/istio-$ISTIO_VERSION/install/kubernetes/helm/istio-init/files

for i in ./crd*yaml; do kubectl apply -f $i; done
```

### Change directory to Istio helm charts, create a tiller service account and re-initialize the service

```
cd ~/istio-$ISTIO_VERSION/install/kubernetes/helm

kubectl apply -f helm-service-account.yaml
helm init --service-account tiller
```

### Validate the Tiller pod is running in kube-system namespace

```
kubectl get pods -n kube-system | grep tiller
```

### Run the following helm template command to generate the yaml file 

```
cd ~/istio-$ISTIO_VERSION
helm template install/kubernetes/helm/istio --name istio \
 --namespace istio-system \
 --values install/kubernetes/helm/istio/values-istio-demo.yaml | \
 kubectl apply -f -
```

### Check pods in istio-system and wait until all are ready
```
kubectl get pods -n istio-system
```

 ### Delete previous installation
```
 cd ~/istio-$ISTIO_VERSION
 helm template install/kubernetes/helm/istio --name istio \
 --namespace istio-system \
 --values install/kubernetes/helm/istio/values-istio-demo.yaml |\
 kubectl delete -f -
```

 ### First, create the custom resource definitions required by Istio

```
cd ~/istio-$ISTIO_VERSION/install/kubernetes/helm
helm install ./istio-init --name istio-init --namespace istio-system
```

### Next, run the helm install command to install istio-demo (permissive mutual TLS)
```
helm install ./istio -f istio/values-istio-demo.yaml \
--name istio --namespace istio-system
```

### Check deployment resources in istio-system.
```
kubectl -n istio-system get deployment
```

### Check pods in istio-system and wait until all are ready
```
kubectl get pods -n istio-system
```

### Uninstall Istio using helm and tiller 
```
helm del --purge istio
helm del --purge istio-init
```

### Install Istio using a demo profile
```
cd ~/istio-$ISTIO_VERSION/
kubectl apply -f install/kubernetes/istio-demo.yaml
```

### Check the version of the istioctl and different Istio modules

```
istioctl version --short
```

### Installing a Load Balancer
### Make sure that the ip_vs kernel module is loaded

```
sudo lsmod | grep ^ip_vs
sudo ipvsadm -ln
sudo lsmod | grep ^ip_vs
```

### Add ip_vs to the module list so that it is loaded automatically on reboot
```
echo "ip_vs" | sudo tee /etc/modules-load.d/ipvs.conf
```

### Label node proxy=true for keepalived
```
kubectl label node osc01.servicemesh.local proxy=true
```

### Install keepalived through a helm chart

```
helm repo add kaal https://servicemeshbook.github.io/keepalived
helm repo update
```

### Grant cluster admin to the default service account in keepalived namespace 

```
kubectl create clusterrolebinding keepalived-cluster-role-binding \
--clusterrole=cluster-admin --serviceaccount=keepalived:default


helm install kaal/keepalived --name keepalived --namespace keepalived \
--set keepalivedCloudProvider.serviceIPRange="192.168.142.248/29" \
--set nameOverride="lb"
```

### Test the readiness and status of pods in keepalived namespace

```
kubectl -n keepalived get pods
```

### Enable Istio for existing application

### Generate modified yaml with sidecar proxy for the bookinfo application
```
cd ~/servicemesh
istioctl kube-inject -f bookinfo.yaml > bookinfo_proxy.yaml
```

### Do a "diff" on original and modified file
```
diff -y bookinfo.yaml bookinfo_proxy.yaml
```

### Deploy modified bookinfo_proxy.yaml to inject a sidecar proxy to the existing bookinfo microservice

```
kubectl -n istio-lab apply -f bookinfo_proxy.yaml
```

### Wait a few minutes for the existing pods to terminate and for the new pods to be ready
```
kubectl -n istio-lab get pods
```

### Enable Istio for new applications

### First delete istio-lab namespace

```
kubectl delete namespace istio-lab
```

### Create istio-lab namespace again and label it using istio-injection=enabled
```
kubectl create namespace istio-lab
kubectl label namespace istio-lab istio-injection=enabled
```

### Deploy the application again.
```
kubectl -n istio-lab apply -f ~/servicemesh/bookinfo.yaml
```

### Check pods
```
kubectl -n istio-lab get pods
```

### Check the current auto scaling for every Istio component

```
kubectl -n istio-system get hpa
```