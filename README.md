# Learn Istio By Examples

Follow the examples here as you read the book.

## Build your own Kubernetes cluster

Follow instructions to build your own Kubernetes environment by going through [https://github.com/servicemeshbook/byok](https://github.com/servicemeshbook/byok).

## Follow Install Istio chapter

The brief instructions are as follows.


### Download Istio

Go to https://istio.io

We will be using Istio version 1.2.2 for the purpose of this branch.

Download using `curl` command.

```
$ cd ~/
$ curl -L https://git.io/getLatestIstio | ISTIO_VERSION=1.2.2 sh -

Downloaded into istio-1.2.2:
bin  install  istio.VERSION  LICENSE  README.md  samples  tools
Add /root/bin/servicemesh/istio-1.2.2/bin to your path; e.g copy paste in your shell and/or ~/.profile:
export PATH="$PATH:/root/bin/servicemesh/istio-1.2.2/bin"
```

Add istio to your path. Edit `.bashrc` and add PATH. Hint: export PATH is shown above.

Your path may be different depending upon the folder in which you downloaded the Istio

```
ISTIO_VERSION=1.2.2
if [ -d ~/istio-${ISTIO_VERSION}/bin ] ; then
  export PATH="$PATH:~/istio-${ISTIO_VERSION}/bin"
fi
```

Source `.bashrc`

```
source ~/.bashrc
```

Run `istioctl verify-install` to make sure that Istio meets all the requirements.

### Install Istio without using mTLS

Makes sure that `ISTIO_VERSION` is set in your `.bashrc`

```
cd ~/istio-${ISTIO_VERSION}/install/kubernetes
kubectl apply -f istio-demo.yaml   

<< Output ommitted>>
service/istio-galley created
service/istio-egressgateway created
service/istio-ingressgateway created
service/grafana created
service/kiali created
service/istio-policy created
service/istio-telemetry created
service/istio-pilot created
service/prometheus created
service/istio-citadel created
service/istio-sidecar-injector created
deployment.extensions/istio-galley created
deployment.extensions/istio-egressgateway created
deployment.extensions/istio-ingressgateway created
deployment.extensions/grafana created
deployment.extensions/kiali created
deployment.extensions/istio-policy created
deployment.extensions/istio-telemetry created
deployment.extensions/istio-pilot created
deployment.extensions/prometheus created
deployment.extensions/istio-citadel created
deployment.extensions/istio-sidecar-injector created
deployment.extensions/istio-tracing created
<< Output ommitted>>

```

Check `istioctl version`

```
$ istioctl version --short
client version: 1.2.2
citadel version: 1.2.2
egressgateway version: 1.2.2
galley version: 1.2.2
ingressgateway version: 1.2.2
pilot version: 1.2.2
policy version: 1.2.2
sidecar-injector version: 1.2.2
telemetry version: 1.2.2
```

Check pod status

```
# kubectl -n istio-system get pods
NAME                                      READY   STATUS      RESTARTS   AGE
grafana-5c45779547-9jxsb                  1/1     Running     0          3m15s
istio-citadel-5bbc997554-8nn4t            1/1     Running     0          3m8s
istio-cleanup-secrets-1.1.3-wc8tw         0/1     Completed   0          3m19s
istio-egressgateway-79df47bcfb-gwzgl      1/1     Running     0          3m17s
istio-galley-5ff6d64c5f-ddpwd             1/1     Running     0          3m18s
istio-grafana-post-install-1.1.3-qxbrl    0/1     Completed   0          3m19s
istio-ingressgateway-5c6bcff97c-wtg9x     1/1     Running     0          3m16s
istio-pilot-69d7cc7c97-84zcd              2/2     Running     0          3m10s
istio-policy-574f8cf96b-x5tml             2/2     Running     5          3m13s
istio-security-post-install-1.1.3-cmtm2   0/1     Completed   0          3m19s
istio-sidecar-injector-549585c8d9-rj9r6   1/1     Running     0          3m7s
istio-telemetry-6c9df8f48b-k4ltv          2/2     Running     5          3m11s
istio-tracing-5fbc94c494-jpdgz            1/1     Running     0          3m7s
kiali-56d95cf466-q8vqt                    1/1     Running     0          3m15s
prometheus-8647cf4bc7-p8b4v               1/1     Running     0          3m9s
```

### Label istio-lab with istio-injection=enabled

```
$ kubectl create ns istio-lab
$ kubectl create clusterrolebinding istio-lab-cluster-role-binding --clusterrole=cluster-admin --serviceaccount=istio-lab:default

$ kubectl label namespace istio-lab istio-injection=enabled
```

#### Simulate Load Balancer

Use `keepalived` Helm chart from : [https://github.com/servicemeshistio/keepalived](https://github.com/servicemeshistio/keepalived)

Download Helm Chart

```console
cd
git clone https://github.com/servicemeshistio/keepalived.git
```

Install Helm Chart

Prerequisites:

`ipvs` module is needed by the vip manager.

Make sure that the ip_vs module is loaded.

```
$ sudo lsmod| grep ^ip_vs
```

If no `ip_vs` module is loaded, install `ipvsadm` 

```console
$ sudo yum -y install ipvsadm

$ sudo ipvsadm -ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn

$ sudo lsmod | grep ^ip_vs
ip_vs                 145497  0
```

Add the module to `/etc/modules-load.d/ipvs.conf` to load the module in case of a reboot.

```console
$ echo "ip_vs" | sudo tee /etc/modules-load.d/ipvs.conf
```

```console
echo Use subnet mask 29 to reserve 6 hosts in 192.168.142.248/29 Class C network.
echo keepalivedCloudProvider.serviceIPRange="192.168.142.248/29"

cd keepalived

helm install . --name keepalived --namespace istio-system \
 --set keepalivedCloudProvider.serviceIPRange="192.168.142.248/29" --tls

```

#### Check load balancer

```console
# kubectl -n istio-system get svc
Name                     TYPE           CLUSTER-IP   EXTERNAL-IP       PORT(S)                                                                                                                                      AGE
grafana                  ClusterIP      10.0.0.65    <none>            3000/TCP                                                                                                                                     86m
istio-citadel            ClusterIP      10.0.0.75    <none>            8060/TCP,15014/TCP                                                                                                                           86m
istio-egressgateway      ClusterIP      10.0.0.244   <none>            80/TCP,443/TCP,15443/TCP                                                                                                                     86m
istio-galley             ClusterIP      10.0.0.90    <none>            443/TCP,15014/TCP,9901/TCP                                                                                                                   86m
istio-ingressgateway     LoadBalancer   10.0.0.229   192.168.142.249   80:31380/TCP,443:31390/TCP,31400:31400/TCP,15029:31730/TCP,15030:31651/TCP,15031:30465/TCP,15032:32041/TCP,15443:31112/TCP,15020:30638/TCP   86m
istio-pilot              ClusterIP      10.0.0.63    <none>            15010/TCP,15011/TCP,8080/TCP,15014/TCP                                                                                                       86m
istio-policy             ClusterIP      10.0.0.88    <none>            9091/TCP,15004/TCP,15014/TCP                                                                                                                 86m
istio-sidecar-injector   ClusterIP      10.0.0.120   <none>            443/TCP                                                                                                                                      86m
istio-telemetry          ClusterIP      10.0.0.45    <none>            9091/TCP,15004/TCP,15014/TCP,42422/TCP                                                                                                       86m
jaeger-agent             ClusterIP      None         <none>            5775/UDP,6831/UDP,6832/UDP                                                                                                                   86m
jaeger-collector         ClusterIP      10.0.0.54    <none>            14267/TCP,14268/TCP                                                                                                                          86m
jaeger-query             ClusterIP      10.0.0.14    <none>            16686/TCP                                                                                                                                    86m
kiali                    ClusterIP      10.0.0.92    <none>            20001/TCP                                                                                                                                    86m
prometheus               ClusterIP      10.0.0.218   <none>            9090/TCP                                                                                                                                     86m
tracing                  ClusterIP      10.0.0.135   <none>            80/TCP                                                                                                                                       86m
zipkin                   ClusterIP      10.0.0.158   <none>            9411/TCP                                                                                                                                     86m

```
### Deploy application

```
$ mkdir -p ~/servicemesh
$ curl -L https://raw.githubusercontent.com/istio/istio/master/samples/bookinfo/platform/kube/bookinfo.yaml -o ~/servicemesh/bookinfo.yaml
```

Deploy application

```console
$ kubectl -n istio-lab apply -f ~/servicemesh/bookinfo.yaml

service/details created
deployment.extensions/details-v1 created
service/ratings created
deployment.extensions/ratings-v1 created
service/reviews created
deployment.extensions/reviews-v1 created
deployment.extensions/reviews-v2 created
deployment.extensions/reviews-v3 created
service/productpage created
deployment.extensions/productpage-v1 created

```

Check pods and wait until all show as `Running`.

```console
# kubectl -n istio-lab get pods
NAME                              READY   STATUS    RESTARTS   AGE
details-v1-bc557b7fc-5vn22        2/2     Running   0          42s
productpage-v1-6597cb5df9-pqx8s   2/2     Running   0          40s
ratings-v1-5c46fc6f85-5tw4n       2/2     Running   0          42s
reviews-v1-69dcdb544-nglxq        2/2     Running   0          42s
reviews-v2-65fbdc9f88-lttjs       2/2     Running   0          41s
reviews-v3-bd8855bdd-85flj        2/2     Running   0          40s
```

Now, you ae ready to follow the chapter to practice the examples.