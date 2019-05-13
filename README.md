# Learn Istio By Examples

Follow the examples here as you read the book.

## Install Istio

There are various methods to use and install Istio.

### Using Public Cloud 

Use Istio as a Managed Service from Cloud Providers

#### IBM Cloud

#### Google Cloud

### Using On-premises or Private Cloud

If using private cloud behind firewall of your enterprise

#### Use Istio Helm Chart by a provider

##### Using IBM Cloud Private

##### Using RedHat OpenShift

#### Use Istio from istio.io

We will use this as an example in this book.

Find out latest version of Istio

```
ISTIO_VERSION=$(curl -L -s https://api.github.com/repos/istio/istio/releases/latest | grep tag_name | sed "s/ *\"tag_name\": *\"\\(.*\\)\",*/\\1/")

# echo $ISTIO_VERSION
1.1.3
```

Find out previous releases if you do not want to use the latest release

```
# curl -L -s https://api.github.com/repos/istio/istio/releases | grep tag_name
```

##### Using Helm

Use a helm chart provided through istio.io

We will work on using istio by manually downloading it.

#### Download Istio

Go to https://istio.io

As writing of this, version 1.1.3 is the latest version.

Download using `curl` command.

```
curl -L https://git.io/getLatestIstio | ISTIO_VERSION=1.1.3 sh -

Downloaded into istio-1.1.3:
bin  install  istio.VERSION  LICENSE  README.md  samples  tools
Add /root/bin/servicemesh/istio-1.1.3/bin to your path; e.g copy paste in your shell and/or ~/.profile:
export PATH="$PATH:/root/bin/servicemesh/istio-1.1.3/bin"
```

Add istio to your path. Edit `.bashrc` and add PATH. Hint: export PATH is shown above.

Your path may be different depending upon the folder in which you downloaded the Istio

```
ISTIO_VERSION=1.1.3
if [ -d ~/istio-${ISTIO_VERSION}/bin ] ; then
  export PATH="$PATH:~/istio-${ISTIO_VERSION}/bin"
fi
```

Source `.bashrc`

```
source ~/.bashrc
```

Check `istioctl version`

```
# istioctl version

version.BuildInfo{Version:"1.1.3", GitRevision:"2b1331886076df103179e3da5dc9077fed59c989", User:"root", Host:"35adf5bb-5570-11e9-b00d-0a580a2c0205", GolangVersion:"go1.10.4", DockerHub:"docker.io/istio", BuildStatus:"Clean", GitTag:"1.1.1"}
```

#### Install Istio without using mTLS

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

#### Label istio-lab with istio-injection=enabled

```
kubectl create ns istio-lab
kubectl label namespace istio-lab istio-injection=enabled
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
lsmod| grep ^ip_vs
```

If no `ip_vs` module is loaded, install `ipvsadm` 

```console
yum -y install ipvsadm

# ipvsadm -ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn

# lsmod | grep ^ip_vs
ip_vs                 145497  0
```

Add the module to `/etc/modules-load.d/ipvs.conf` to load the module in case of a reboot.

```console
echo "ip_vs" > /etc/modules-load.d/ipvs.conf
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

#### Run bookinfo with proxy

How to enable Istio for an existing application.

Method 1: Use istioctl to generate the modified yaml that will include proxy cars.

How to enable Istio for new application application.

Let's delete the earlier `istio-lab` name space. 

```console
kubectl delete ns istio-lab
```

Create name space and label for the web hooks to create proxy sidecar

```console
kubectl create ns istio-lab

kubectl label namespace istio-lab istio-injection=enabled
```

Deploy application

```console
kubectl -n istio-lab apply -f bookinfo.yaml 

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

### Run application

Multiple cases:

* Run using pod's internal IP address

  Find Pod's IP address for `productpage`  

  ```console
  # kubectl -n istio-lab get pods -o wide
  NAME                              READY   STATUS    RESTARTS   AGE   IP             NODE              NOMINATED NODE
  details-v1-bc557b7fc-99d4b        2/2     Running   0          21s   10.1.230.230   192.168.142.101   <none>
  productpage-v1-6597cb5df9-jqhcq   2/2     Running   0          18s   10.1.230.246   192.168.142.101   <none>
  ratings-v1-5c46fc6f85-pvh9l       2/2     Running   0          21s   10.1.230.231   192.168.142.101   <none>
  reviews-v1-69dcdb544-6dbpz        2/2     Running   0          21s   10.1.230.214   192.168.142.101   <none>
  reviews-v2-65fbdc9f88-4bpl5       2/2     Running   0          20s   10.1.230.247   192.168.142.101   <none>
  reviews-v3-bd8855bdd-zb5zk        2/2     Running   0          19s   10.1.230.201   192.168.142.101   <none>
  ```

  In above case, it is `10.1.230.246`. This IP address will be different for you.

  Test the service using pod's IP address. Change IP address as per your output.

  ```console
  curl -s http://10.1.230.246:9080 | grep title
  <title>Simple Bookstore App</title>
  ```

* Run using internal service name's IP address - requires server access using local service name

  ```console
  # kubectl -n istio-lab get svc
  NAME          TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
  details       ClusterIP   10.0.0.181   <none>        9080/TCP   102s
  productpage   ClusterIP   10.0.0.20    <none>        9080/TCP   100s
  ratings       ClusterIP   10.0.0.66    <none>        9080/TCP   102s
  reviews       ClusterIP   10.0.0.45    <none>        9080/TCP   102s
  ```
  The productpage service address is `10.0.0.20`. This IP address might be different for you.

  Test the service and use IP address as per your service IP address.

  ```console
  # curl -s http://10.0.0.20:9080 | grep title
    <title>Simple Bookstore App</title>
  ```

  The service IP address does not change for the life time of service. However, the pod address may change depending upon the node on which it gets scheduled.

  The connection between pod's IP address and service IP address is through the endpoints.

  ```console
  # kubectl -n istio-lab get ep
  NAME          ENDPOINTS                                               AGE
  details       10.1.230.230:9080                                       2m48s
  productpage   10.1.230.246:9080                                       2m46s
  ratings       10.1.230.231:9080                                       2m48s
  reviews       10.1.230.201:9080,10.1.230.214:9080,10.1.230.247:9080   2m48s
  ```
  Notice that the productpage service endpoint is `10.1.230.246` which is the pod's IP address.

  The service name can also be used provided Kubernetes internal DNS server is on the search list.

  Check the IP address of the internal service name.

  ```console
  # dig +short productpage.istio-lab.svc.cluster.local @10.0.0.10
  10.0.0.20
  ```

  This IP address `10.0.0.20` is the IP address of the FQDN of the service name.

  Open a browser from inside the VM and try `http://10.0.0.20:9080/` and `http://10.0.0.20:9080/` and you should be able to see the product page.

* Run using a node port - requires server IP address - Run from outside the cluster but requires IP address for the cluster

  You can also view the service web page from outside VM but within the firewall of an enterprise by using the server's IP address. This would require changing the service from `ClusterIP` to the `NodePort`.

  ```console
  kubectl -n istio-lab edit svc productpage
  ```

  And change the type from `ClusterIP` to `NodePort` 

  From:

  ```console
    selector:
    app: productpage
  sessionAffinity: None
  type: ClusterIP

  ```

  To:
  ```console
    selector:
    app: productpage
  sessionAffinity: None
  type: NodePort
  ```

  And save the file and again check the services.

  ```console
  # kubectl -n istio-lab get svc
  NAME          TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)          AGE
  details       ClusterIP   10.0.0.181   <none>        9080/TCP         10m
  productpage   NodePort    10.0.0.20    <none>        9080:30306/TCP   10m
  ratings       ClusterIP   10.0.0.66    <none>        9080/TCP         10m
  reviews       ClusterIP   10.0.0.45    <none>        9080/TCP         10m
  ```

  Notice the `NodePort` for the`productpage` service is `30306`. This port number will be different in your case.

  Now, you can open a browser from outside the VM (or cluster in reality) and access the `productpage`. 

  Find out the IP address of the VM or master node of the ICP (or Kubernetes) cluster.

  ```console
  # kubectl get nodes
  NAME              STATUS   ROLES                                 AGE   VERSION
  192.168.142.101   Ready    etcd,management,master,proxy,worker   29d   v1.12.4+icp
  ```

  The IP address of the node is `192.168.142.101` and the node port is `30306`. Your NodePort might be different.

  If you open the browser to http://192.168.142.101:30306 from outside the VM, you should be able to access the service.

* Run using a load balancer IP address from outside the cluster

  It may so happen that the ICP or Kubernetes cluster is behind the firewall and not accessible to the outside world. In that case, we may have an external load balancer that we can use to route the traffic from internal service to the outside.

  We are simulating an external load balancer using `keepalived` but using IP address on the same subnet range of the VM. In reality, you will get an external IP addresses against the `istio-ingress-gatewaey` if an external load balancer is configured.

  In our case, find out the external IP address.

  ```console
  # kubectl -n istio-system get svc | grep ingressgateway
  istio-ingressgateway     LoadBalancer   10.0.0.223   192.168.142.250   80:31380/TCP,443:31390/TCP,31400:31400/TCP,15029:32030/TCP,15030:30237/TCP,15031:30824/TCP,15032:30965/TCP,15443:31507/TCP,15020:31796/TCP   172m
  ```

  Notice that the external IP address assigned to `istio-ingressgateway` is `192.168.142.250`. We can access the service web page by using `http://192.168.142.250`. This IP address could be different for you.

  Try accessing the web page and you will notice that the page did not work.

  Now, we now to create a gateway and virtual services so that the request can be routed properly when it comes to the 

  Create Istio Gateway and Virtual Service `istio-ingressgateway`

  ### Istio Gateway and Virtual Service Definition

  Run YAML `01-create-gateway-virtual-service` has definition for a gateway and virtual service.

  ```yaml
  apiVersion: networking.istio.io/v1alpha3
  kind: Gateway
  metadata:
    name: bookinfo-gateway
  spec:
    selector:
      istio: ingressgateway # use istio default controller
    servers:
    - port:
        number: 80
        name: http
        protocol: HTTP
      hosts:
      - "*"
  ---
  apiVersion: networking.istio.io/v1alpha3
  kind: VirtualService
  metadata:
    name: bookinfo
  spec:
    hosts:
    - "*"
    gateways:
    - bookinfo-gateway
    http:
    - match:
      - uri:
          exact: /productpage
      - uri:
          prefix: /static      
      - uri:
          exact: /login
      - uri:
          exact: /logout
      - uri:
          prefix: /api/v1/products
      route:
      - destination:
          host: productpage
          port:
            number: 9080
  ```

  ### Create Gateway and Virtual Service

  ```console
  kubectl -n istio-lab apply -f 01-create-gateway-virtual-service.yaml 

  gateway.networking.istio.io/bookinfo-gateway created
  virtualservice.networking.istio.io/bookinfo created
  ```

  Define gateway - how it is working

  Define Virtual Service - how it is working

  Check Gateway and Virtual Service

  ```console
  # kubectl -n istio-lab get gateway
  NAME               AGE
  bookinfo-gateway   2m

  # kubectl -n istio-lab get virtualservice
  NAME       GATEWAYS             HOSTS   AGE
  bookinfo   [bookinfo-gateway]   [*]     3m
  ```

  We are redirecting url path `/productpage` to internal service `productpage` at port `9080`.

  Try accessing the url: [http://192.168.142.250/productpage](http://192.168.142.250/productpage)

  If the IP address is defined in the DNS server, the internal service can be accessed using a domain name. 

## Destination rules

DestinationRule is defining subsets for every service within BookInfo. Subset is using pod labels through which routing rules can be defined.

Run script `kubectl -n istio-lab apply -f 02-create-destination-rules.yaml`

Contents of 02-create-destination-rules.yaml

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: productpage
spec:
  host: productpage
  subsets:
  - name: v1
    labels:
      version: v1
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
  - name: v3
    labels:
      version: v3
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: ratings
spec:
  host: ratings
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
  - name: v2-mysql
    labels:
      version: v2-mysql
  - name: v2-mysql-vm
    labels:
      version: v2-mysql-vm
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: details
spec:
  host: details
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
---
```
## Virtual Service using subsets

What is a virtual service and what different things can be accomplished?

Run script `kubectl -n istio-lab apply -f 03-create-reviews-virtual-service.yaml`

Contents of `03-create-reviews-virtual-service.yaml`

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: productpage
spec:
  hosts:
  - productpage
  http:
  - route:
    - destination:
        host: productpage
        subset: v1
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v2
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - route:
    - destination:
        host: ratings
        subset: v1
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: details
spec:
  hosts:
  - details
  http:
  - route:
    - destination:
        host: details
        subset: v1
---
```
## Identity Based Traffic Routing

Change route configuration so all traffic from a specific user "Jason" will be routed to reviews-v2

Run script `kubectl -n istio-lab apply -f 04-create-reviews-user-virtual-service.yaml`


## Traffic Shifting

Gradually migrate traffic from one microservice to another based on weight. For instance, send 50% of traffic from reviews-v1 to reviews-v3. 

Validate Bookinfo virtual service for productpage, ratings, reviews and details is up by running `kubectl get virtualservice` OR `k get vs`

```console
[root@osc01 (istio-lab)istio-scripts]# k get vs
NAME          GATEWAYS             HOSTS           AGE
bookinfo      [bookinfo-gateway]   [*]             1h
details                            [details]       1h
productpage                        [productpage]   1h
ratings                            [ratings]       1h
reviews                            [reviews]       1h
```
Apply the following YAML to transfer 50% traffic from reviews-1 to reviews-3.

```console
[root@osc01 (istio-lab)istio-scripts]# kubectl -n istio-lab apply -f 05-apply-weight-based-routing.yaml
virtualservice.networking.istio.io/reviews configured
```

Confirm the rule was replaced for reviews microservice by running `kubectl get virtualservice reviews -o yaml` OR `k get vs reviews -o yaml`

```console
[root@osc01 (istio-lab)istio-scripts]# k get vs reviews -o yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"networking.istio.io/v1alpha3","kind":"VirtualService","metadata":{"annotations":{},"name":"reviews","namespace":"istio-lab"},"spec":{"hosts":["reviews"],"http":[{"route":[{"destination":{"host":"reviews","subset":"v1"},"weight":50},{"destination":{"host":"reviews","subset":"v3"},"weight":50}]}]}}
  creationTimestamp: 2019-04-16T02:05:31Z
  generation: 1
  name: reviews
  namespace: istio-lab
  resourceVersion: "50804"
  selfLink: /apis/networking.istio.io/v1alpha3/namespaces/istio-lab/virtualservices/reviews
  uid: 1cfbe884-5fec-11e9-9989-00505632f6a0
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v1
      weight: 50
    - destination:
        host: reviews
        subset: v3
      weight: 50
```
Submit 100% of the traffic to reviews-3 microservice.

```Console
[root@osc01 (istio-lab)istio-scripts]# kubectl -n istio-lab apply -f 06-apply-total-weight-based-routing.yaml
virtualservice.networking.istio.io/reviews configured
```


## Fault Injection

Test microservice resiliency by injecting faults

### Injecting HTTP delay fault

Inject a 7 second delay between reviews-v2 and ratings microservice for user "Jason". This is an intentional because the there are hardcode timeouts in the microservice. 

Run script 
```console
[root@osc01 (istio-lab)istio-scripts]# kubectl -n istio-lab apply -f 07-inject-fault-user-trafficdelay.yaml
virtualservice.networking.istio.io/ratings configured
```

After 7 seconds, the reviews section will display an error message:
```
Error fetching product reviews!
Sorry, product reviews are currently unavailable for this book.
```
There are hardcoded timeouts between `productpage` and `reviews` page at 6 seconds.
There are hardcoded tiemouts between `reviews` and `ratings` page at 10 seconds. 

Introducing the new 7 second delay, the `productpage` times out premarturely. Fault injection helps identify such anomalies without impacting end users. 

If you logout of "Jason" user, the error will go away. 

### Inject HTTP abort fault

To test microservice resiliency, introduce HTTP abort fault. 

Run script
```console
[root@osc01 (istio-lab)istio-scripts]# kubectl -n istio-lab apply -f 08-inject-fault-user-aborttest.yaml
virtualservice.networking.istio.io/ratings configured
```

Validate the `ratings` virtual service has applied the changes by running `k get vs ratings -o yaml`

The HTTP abort to the ratings microservice for user "Jason" will load "Ratings service is not available". 

If you logout of "Jason" user, the error will go away. 

## TCP Traffic Shifting 

Migrate TCP traffice from one version of a microservice to another. To accomplish this goal, configure a sequence of rules that route percentage of TCp traffic from one service to another. 

In this task, we will deploy the tcp-echo-v1 microserive. Send 100% to this new service and route 20% of the TCP traffic to tcp-echo-v2

Deploy the `tcp-echo` microservice:

```console
[root@osc01 (istio-lab)istio-scripts]# kubectl -n istio-lab apply -f 09-create-tcpecho-service.yaml
service/tcp-echo created
deployment.extensions/tcp-echo-v1 created
deployment.extensions/tcp-echo-v2 created
```

Next, enable all TCP traffic to tcp-echo-v1 microservice:

Run script:
```console
[root@osc01 (istio-lab)istio-scripts]# kubectl -n istio-lab apply -f 10-apply-tcpecho-v1-routealltraffic.yaml
gateway.networking.istio.io/tcp-echo-gateway created
destinationrule.networking.istio.io/tcp-echo-destination created
virtualservice.networking.istio.io/tcp-echo created
```

```
Validate the `tcp-echo` microservices are running:

```console
[root@osc01 (istio-lab)istio-scripts]# kgp
NAME                              READY   STATUS    RESTARTS   AGE
details-v1-8548c7db94-6bg57       2/2     Running   6          45h
productpage-v1-84cc5ff8d4-5f2m5   2/2     Running   6          45h
ratings-v1-bb7c878-ncpjh          2/2     Running   6          45h
reviews-v1-7d56484db5-nb4vj       2/2     Running   6          45h
reviews-v2-65f775cff6-45wx6       2/2     Running   6          45h
reviews-v3-75594d8d75-gpq4s       2/2     Running   6          45h
tcp-echo-v1-8694f878bf-xbt5h      2/2     Running   0          7m3s
tcp-echo-v2-777dfc4954-dzfrv      2/2     Running   0          7m3s
```

Next, obtain the INGRESS_HOST value so traffic can be sent to the microservice.

Run the following command to get the `Ingress Host` variable:

```console
export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="tcp")].port}')
```
Find out the value:
```console
[root@osc01 (istio-system)istio-scripts]# echo $INGRESS_PORT
31400
```
Validate the Ingress host, port are available:

```console
[root@osc01 (istio-lab)istio-scripts]# export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
[root@osc01 (istio-lab)istio-scripts]# export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="tcp")].port}')
[root@osc01 (istio-lab)istio-scripts]# echo $INGRESS_HOST
192.168.142.250
[root@osc01 (istio-lab)istio-scripts]# echo $INGRESS_PORT
31400
```

Send some TCP traffic to the tcp-echo microservice:

```console
[root@osc01 (istio-lab)istio-scripts]# for i in {1..10}; do \
> docker run -e INGRESS_HOST=$INGRESS_HOST -e INGRESS_PORT=$INGRESS_PORT -it --rm busybox sh -c "(date; sleep 1) | nc $INGRESS_HOST $INGRESS_PORT"; \
> done
one Wed Apr 17 02:52:06 UTC 2019
one Wed Apr 17 02:52:08 UTC 2019
one Wed Apr 17 02:52:10 UTC 2019
one Wed Apr 17 02:52:12 UTC 2019
one Wed Apr 17 02:52:13 UTC 2019
one Wed Apr 17 02:52:15 UTC 2019
one Wed Apr 17 02:52:17 UTC 2019
one Wed Apr 17 02:52:19 UTC 2019
one Wed Apr 17 02:52:20 UTC 2019
one Wed Apr 17 02:52:22 UTC 2019
```
The prefix of `one` signifies that all traffic was routed to the v1 of tcp-echo microservice.

Next, setup environment to send 20% of the traffic from tcp-echo-v1 to tcp-echo-v2.

Run script:
```console
[root@osc01 (istio-lab)istio-scripts]# kubectl -n istio-lab apply -f 11-apply-tcpecho-v1-route20traffic.yaml
virtualservice.networking.istio.io/tcp-echo configured
```

Send 20% TCP traffic:

```console
[root@osc01 (istio-lab)istio-scripts]# for i in {1..10}; do \
> docker run -e INGRESS_HOST=$INGRESS_HOST -e INGRESS_PORT=$INGRESS_PORT -it --rm busybox sh -c "(date; sleep 1) | nc $INGRESS_HOST $INGRESS_PORT"; \
> done
two Wed Apr 17 02:59:42 UTC 2019
one Wed Apr 17 02:59:44 UTC 2019
one Wed Apr 17 02:59:45 UTC 2019
one Wed Apr 17 02:59:47 UTC 2019
two Wed Apr 17 02:59:49 UTC 2019
one Wed Apr 17 02:59:50 UTC 2019
one Wed Apr 17 02:59:52 UTC 2019
one Wed Apr 17 02:59:54 UTC 2019
one Wed Apr 17 02:59:56 UTC 2019
one Wed Apr 17 02:59:57 UTC 2019
```

2 out of the 10 timestamps have a `two`, meaning that 80% of all TCP traffic was routed to v1, whereas 20% was routed to v2.

## Request Timeouts

HTTP request timeouts can be specified using the timeout filed within a route rule within YAML. In default, this is set at 15 seconds. In this exercise, the timeout will be 1 second. 

To see this occur, 2 second artificial delay in any calls to the `ratings ` service will be introduced. 

Route requests to reviews-v2
```console
[root@osc01 (istio-lab)istio-scripts]# kubectl -n istio-lab apply -f 12-create-reviews-route-requests.yaml
virtualservice.networking.istio.io/reviews configured
```

Next, add a 2 second delay to calls to rating service
```console
[root@osc01 (istio-lab)istio-scripts]# kubectl -n istio-lab apply -f 13-apply-ratings-2secdelay.yaml
virtualservice.networking.istio.io/ratings configured
```
Go to BookInfo web console and refresh the page, make sure to logout from any users. 

The application will be working normally, but there is a 2 second delay with every refresh. To validate the time, go to `developers tools` in Chrome/Firefox browser and validate the time under the `network` tab.

Next, add a half a second request timeout for calls to reviews service.

```console
[root@osc01 (istio-lab)istio-scripts]# kubectl -n istio-lab apply -f 14-apply-reviews-0.5sectimeout.yaml
virtualservice.networking.istio.io/reviews configured
```
With this update, the returns are in 1 second instead of 2 (leverage Browser - Developer tools to validate) and the reviews are unavailable.

Reason why the responses take 1 second, even though a 0.5 second timeout is configured, is because there is a hard coded re-try within the productpage microservice so its saying the reviews service is timing out twice before returning. 

## Circuit Breaker

In this exercise, this shows you how to configure circuit breaking for connections, requests and outlier detection and intentionally `tripping` the circuit breaker.

Circuit breaking creates resilient microservices and allows application writes that limit failures, latency spikes and other network pecularities.

Before getting started, deploy the `HTTPbin` microservice:

```console
[root@osc01 (istio-lab)istio-scripts]# kubectl -n istio-lab apply -f 15-create-httpbin-service.yamlservice/httpbin created
deployment.extensions/httpbin created
```

Validate the service pods for `HTTPbin` are available:

```console
[root@osc01 (istio-lab)istio-scripts]# kgp
NAME                              READY   STATUS    RESTARTS   AGE
details-v1-8548c7db94-6bg57       2/2     Running   6          46h
httpbin-5fc7cf895d-84nl4          2/2     Running   0          10m
productpage-v1-84cc5ff8d4-5f2m5   2/2     Running   6          46h
ratings-v1-bb7c878-ncpjh          2/2     Running   6          46h
reviews-v1-7d56484db5-nb4vj       2/2     Running   6          46h
reviews-v2-65f775cff6-45wx6       2/2     Running   6          46h
reviews-v3-75594d8d75-gpq4s       2/2     Running   6          46h
tcp-echo-v1-8694f878bf-xbt5h      2/2     Running   0          81m
tcp-echo-v2-777dfc4954-dzfrv      2/2     Running   0          81m
```

Create an `HTTPbin` destination rule.

Note: if the existing istio install has `mTLS` authentication enabled,define a `TLS mode: MUTUAL` to the `HTTPbin` destination rule YAML.

```console
[root@osc01 (istio-lab)istio-scripts]# kubectl -n istio-lab apply -f 16-create-httpbin-destination-rule.ymal
destinationrule.networking.istio.io/httpbin created
```

Validate the destination rule was created:

```console
[root@osc01 (istio-lab)istio-scripts]# k get dr httpbin
NAME      HOST      AGE
httpbin   httpbin   2m
```

Next, create a client called `fortio` with sidecar proxy that will send traffic to `HTTPbin` service. This is primiarily to load test the connections, concurrency and delays for all HTTP calls. 

`fortio` will also trip the circuit breaker policies that are defined in the `HTTPbin` destination rule.

```console
[root@osc01 (istio-lab)istio-scripts]# kubectl apply -f <(istioctl kube-inject -f 17-create-client-fortio.yaml)
service/fortio created
deployment.apps/fortio-deploy created
```

Once deployed, login to the fortio client pod:

```console
[root@osc01 (istio-lab)istio-scripts]# FORTIO_POD=$(kubectl get pod | grep fortio | awk '{ print $1 }')
```

Use `fortio` to call `HTTPbin` service through `-curl` to indicate that you want to make 1 call:

```console
[root@osc01 (istio-lab)istio-scripts]# kubectl exec -it $FORTIO_POD  -c fortio /usr/bin/fortio -- load -curl  http://httpbin:8000/get
HTTP/1.1 200 OK
server: envoy
date: Wed, 17 Apr 2019 03:55:20 GMT
content-type: application/json
content-length: 371
access-control-allow-origin: *
access-control-allow-credentials: true
x-envoy-upstream-service-time: 2

{
  "args": {},
  "headers": {
    "Content-Length": "0",
    "Host": "httpbin:8000",
    "User-Agent": "fortio.org/fortio-1.3.1",
    "X-B3-Parentspanid": "616a055736063b89",
    "X-B3-Sampled": "1",
    "X-B3-Spanid": "8b1ef9801f095f37",
    "X-B3-Traceid": "1e9f662f0206c78d616a055736063b89"
  },
  "origin": "127.0.0.1",
  "url": "http://httpbin:8000/get"
}
```

Now that the `fortio` client pod can make successful calls to the `HTTPbin` service, let's trip the circuit breaker.

Within the `HTTPbin` destination rule: 
For TCP, there is 1 `maxConnections`
For HTTP, there is 1 `http1MaxPendingRequests`

What this means is if you exceed 1 connection and request concurrently, the istio-proxy will show failures when the circuit is opened for more-than-1 connections and requests.

Call the `HTTPbin` service with 2 (not 1) conncurent connections and send 20 (not 1) requests.

```console
[root@osc01 (istio-lab)istio-scripts]# kubectl exec -it $FORTIO_POD  -c fortio /usr/bin/fortio -- load -c 2 -qps 0 -n 20 -loglevel Warning http://httpbin:8000/get
04:03:46 I logger.go:97> Log level is now 3 Warning (was 2 Info)
Fortio 1.3.1 running at 0 queries per second, 8->8 procs, for 20 calls: http://httpbin:8000/get
Starting at max qps with 2 thread(s) [gomax 8] for exactly 20 calls (10 per thread + 0)
04:03:46 W http_client.go:679> Parsed non ok code 503 (HTTP/1.1 503)
04:03:46 W http_client.go:679> Parsed non ok code 503 (HTTP/1.1 503)
Ended after 77.394694ms : 20 calls. qps=258.42
Aggregated Function Time : count 20 avg 0.00760278 +/- 0.006768 min 0.001263203 max 0.027271238 sum 0.152055601
# range, mid point, percentile, count
>= 0.0012632 <= 0.002 , 0.0016316 , 5.00, 1
> 0.002 <= 0.003 , 0.0025 , 10.00, 1
> 0.003 <= 0.004 , 0.0035 , 25.00, 3
> 0.004 <= 0.005 , 0.0045 , 30.00, 1
> 0.005 <= 0.006 , 0.0055 , 55.00, 5
> 0.006 <= 0.007 , 0.0065 , 70.00, 3
> 0.007 <= 0.008 , 0.0075 , 75.00, 1
> 0.008 <= 0.009 , 0.0085 , 90.00, 3
> 0.025 <= 0.0272712 , 0.0261356 , 100.00, 2
# target 50% 0.0058
# target 75% 0.008
# target 90% 0.009
# target 99% 0.0270441
# target 99.9% 0.0272485
Sockets used: 4 (for perfect keepalive, would be 2)
Code 200 : 18 (90.0 %)
Code 503 : 2 (10.0 %)
Response Header Sizes : count 20 avg 207.1 +/- 69.03 min 0 max 231 sum 4142
Response Body/Total Sizes : count 20 avg 564.4 +/- 110.3 min 217 max 602 sum 11288
All done 20 calls (plus 0 warmup) 7.603 ms avg, 258.4 qps
```

Overall, almost all requests made it through where the istio-proxy has some leniency:

```console
Code 200 : 18 (90.0 %)
Code 503 : 2 (10.0 %)
```

Call the `HTTPbin` service with 3 (not 2) conncurent connections and send 30 (not 20) requests.

```console
[root@osc01 (istio-lab)istio-scripts]# kubectl exec -it $FORTIO_POD  -c fortio /usr/bin/fortio -- load -c 3 -qps 0 -n 30 -loglevel Warning http://httpbin:8000/get
04:06:58 I logger.go:97> Log level is now 3 Warning (was 2 Info)
Fortio 1.3.1 running at 0 queries per second, 8->8 procs, for 30 calls: http://httpbin:8000/get
Starting at max qps with 3 thread(s) [gomax 8] for exactly 30 calls (10 per thread + 0)
04:06:58 W http_client.go:679> Parsed non ok code 503 (HTTP/1.1 503)
04:06:58 W http_client.go:679> Parsed non ok code 503 (HTTP/1.1 503)
04:06:58 W http_client.go:679> Parsed non ok code 503 (HTTP/1.1 503)
04:06:58 W http_client.go:679> Parsed non ok code 503 (HTTP/1.1 503)
04:06:58 W http_client.go:679> Parsed non ok code 503 (HTTP/1.1 503)
04:06:58 W http_client.go:679> Parsed non ok code 503 (HTTP/1.1 503)
04:06:58 W http_client.go:679> Parsed non ok code 503 (HTTP/1.1 503)
04:06:58 W http_client.go:679> Parsed non ok code 503 (HTTP/1.1 503)
04:06:58 W http_client.go:679> Parsed non ok code 503 (HTTP/1.1 503)
04:06:58 W http_client.go:679> Parsed non ok code 503 (HTTP/1.1 503)
04:06:58 W http_client.go:679> Parsed non ok code 503 (HTTP/1.1 503)
04:06:58 W http_client.go:679> Parsed non ok code 503 (HTTP/1.1 503)
Ended after 57.799732ms : 30 calls. qps=519.03
Aggregated Function Time : count 30 avg 0.0038989608 +/- 0.003022 min 0.000331027 max 0.01162842 sum 0.116968824
# range, mid point, percentile, count
>= 0.000331027 <= 0.001 , 0.000665514 , 30.00, 9
> 0.001 <= 0.002 , 0.0015 , 36.67, 2
> 0.002 <= 0.003 , 0.0025 , 43.33, 2
> 0.003 <= 0.004 , 0.0035 , 60.00, 5
> 0.004 <= 0.005 , 0.0045 , 66.67, 2
> 0.005 <= 0.006 , 0.0055 , 70.00, 1
> 0.006 <= 0.007 , 0.0065 , 76.67, 2
> 0.007 <= 0.008 , 0.0075 , 93.33, 5
> 0.008 <= 0.009 , 0.0085 , 96.67, 1
> 0.011 <= 0.0116284 , 0.0113142 , 100.00, 1
# target 50% 0.0034
# target 75% 0.00675
# target 90% 0.0078
# target 99% 0.0114399
# target 99.9% 0.0116096
Sockets used: 13 (for perfect keepalive, would be 3)
Code 200 : 18 (60.0 %)
Code 503 : 12 (40.0 %)
Response Header Sizes : count 30 avg 138 +/- 112.7 min 0 max 230 sum 4140
Response Body/Total Sizes : count 30 avg 449.66667 +/- 185.5 min 217 max 601 sum 13490
All done 30 calls (plus 0 warmup) 3.899 ms avg, 519.0 qps
[root@osc01 (istio-lab)istio-scripts]#
```

As the number of connections and requests went up, the circuit breaker behavior is expected. Only 60% of the requests succeeded as the remaining 40% are trapped by the circuit breaker.

```console
Code 200 : 18 (60.0 %)
Code 503 : 12 (40.0 %)
```

Query the `istio-proxy` by logging into the `fortio` pod to see further stats on circuit breaker:

```console
[root@osc01 (istio-lab)istio-scripts]# kubectl exec -it fortio-deploy-75f598c8dd-mhjt8  -c istio-proxy  -- sh -c 'curl localhost:15000/stats' | grep httpbin | grep pending
cluster.outbound|8000||httpbin.istio-lab.svc.cluster.local.circuit_breakers.default.rq_pending_open: 0
cluster.outbound|8000||httpbin.istio-lab.svc.cluster.local.circuit_breakers.high.rq_pending_open: 0
cluster.outbound|8000||httpbin.istio-lab.svc.cluster.local.upstream_rq_pending_active: 0
cluster.outbound|8000||httpbin.istio-lab.svc.cluster.local.upstream_rq_pending_failure_eject: 0
cluster.outbound|8000||httpbin.istio-lab.svc.cluster.local.upstream_rq_pending_overflow: 21
cluster.outbound|8000||httpbin.istio-lab.svc.cluster.local.upstream_rq_pending_total: 81
```



Through the `upstream_rq_pending_overflow` value, 21 calls have been flagged for circuit breaking for service `HTTPbin` by client `fortio`.

## Mirroring

This task demonstrates traffic mirroring, also called shadowing, capabilities. This allows teams to bring production changes with little risk because mirroring sends copy of live traffic to a mirrored service. 

In this exercise, force all `HTTPbin` service traffic to version 1. Then, apply a rule to mirror a portion of that traffic to version 2.

Deploy `HTTPbin` service, version 1

```console
[root@osc01 (istio-lab)istio-scripts]# kubectl -n istio-lab apply -f 18-deploy-httpbin-v1.yaml
deployment.extensions/httpbin-v1 created
```

Deploy `HTTPbin` service, version 2

```console
[root@osc01 (istio-lab)istio-scripts]# kubectl -n istio-lab apply -f 19-deploy-httpbin-v2.yaml
deployment.extensions/httpbin-v2 created
```

Validate the two version of `HTTPbin` are up and running

```console
[root@osc01 (istio-lab)istio-scripts]# kgp
NAME                              READY   STATUS    RESTARTS   AGE
details-v1-8548c7db94-6bg57       2/2     Running   6          47h
fortio-deploy-75f598c8dd-mhjt8    2/2     Running   0          50m
httpbin-5fc7cf895d-84nl4          2/2     Running   0          65m
httpbin-v1-6569dfb499-r7xhh       2/2     Running   0          4m29s
httpbin-v2-67c5fc7ffb-9qr7k       2/2     Running   0          57s
productpage-v1-84cc5ff8d4-5f2m5   2/2     Running   6          47h
ratings-v1-bb7c878-ncpjh          2/2     Running   6          47h
reviews-v1-7d56484db5-nb4vj       2/2     Running   6          47h
reviews-v2-65f775cff6-45wx6       2/2     Running   6          47h
reviews-v3-75594d8d75-gpq4s       2/2     Running   6          47h
tcp-echo-v1-8694f878bf-xbt5h      2/2     Running   0          136m
tcp-echo-v2-777dfc4954-dzfrv      2/2     Running   0          136m
```

Deploy and start the `HTTPbin` sleep service extension to leverage curl command for load

```console
[root@osc01 (istio-lab)istio-scripts]# kubectl -n istio-lab apply -f 20-deploy-httpbin-sleep.yaml
deployment.extensions/sleep created

```

By default, kubernetes will load balance both version of `HTTPbin` service. To change that behavior, all traffic will be routed to `HTTPbin` version 1 service.

Note: if the existing istio install has `mTLS` authentication enabled,define a `TLS mode: MUTUAL` to the `HTTPbin` destination rule YAML.

```console
[root@osc01 (istio-lab)istio-scripts]# kubectl -n istio-lab apply -f 21-route-httpbin-alltraffic-v1.yaml
virtualservice.networking.istio.io/httpbin created
destinationrule.networking.istio.io/httpbin configured
```

Send some traffic from `HTTPbin` version 1 to sleep pod

```console
[root@osc01 (istio-lab)istio-scripts]# export SLEEP_POD=$(kubectl get pod -l app=sleep -o jsonpath={.items..metadata.name})
[root@osc01 (istio-lab)istio-scripts]# kubectl exec -it $SLEEP_POD -c sleep -- sh -c 'curl  http://httpbin:8000/headers' | python -m json.tool
{
    "headers": {
        "Accept": "*/*",
        "Content-Length": "0",
        "Host": "httpbin:8000",
        "User-Agent": "curl/7.35.0",
        "X-B3-Parentspanid": "db31aa45c290a85f",
        "X-B3-Sampled": "1",
        "X-B3-Spanid": "7a433da7969422cd",
        "X-B3-Traceid": "df7ce1045ca39ebedb31aa45c290a85f"
    }
}
```
Check the logs within `HTTPbin` v1 and v2 pods to showcase log entries exist in v1 because all traffic was routed there, and nothing in v2. 

Note: Run two bash terminals for both `HTTPbin` versions to see the logs when the curl command from above is executed.

Within `HTTPbin` version 1, the following output shows the following:

```console
[root@osc01 (istio-lab)istio-scripts]# k log -f httpbin-v1-6569dfb499-r7xhh -c httpbin
log is DEPRECATED and will be removed in a future version. Use logs instead.
[2019-04-18 01:49:29 +0000] [1] [INFO] Starting gunicorn 19.9.0
[2019-04-18 01:49:29 +0000] [1] [INFO] Listening at: http://0.0.0.0:80 (1)
[2019-04-18 01:49:29 +0000] [1] [INFO] Using worker: sync
[2019-04-18 01:49:29 +0000] [8] [INFO] Booting worker with pid: 8
127.0.0.1 - - [18/Apr/2019:02:46:20 +0000] "GET /headers HTTP/1.1" 200 303 "-" "curl/7.35.0"
```

Within v2, since no traffic is being passed here, the logs don't show anything.

```console
[root@osc01 (istio-lab)istio-scripts]# k log -f httpbin-v2-67c5fc7ffb-9qr7k -c httpbin
log is DEPRECATED and will be removed in a future version. Use logs instead.
[2019-04-18 01:49:29 +0000] [1] [INFO] Starting gunicorn 19.9.0
[2019-04-18 01:49:29 +0000] [1] [INFO] Listening at: http://0.0.0.0:80 (1)
[2019-04-18 01:49:29 +0000] [1] [INFO] Using worker: sync
[2019-04-18 01:49:29 +0000] [8] [INFO] Booting worker with pid: 8
```

For the next exercise, we will mirror 100% of traffic that is going to `HTTPbin` version 1 for version 2. 

```console
[root@osc01 (istio-lab)istio-scripts]# kubectl apply -f 22-route-httpbin-alltraffic-v2.yaml -n istio-lab
virtualservice.networking.istio.io/httpbin configured
```

Send the traffic to `HTTPbin` version 1.

```console
[root@osc01 (istio-lab)istio-scripts]# kubectl exec -it $SLEEP_POD -c sleep -- sh -c 'curl  http://httpbin:8000/headers' | python -m json.tool
{
    "headers": {
        "Accept": "*/*",
        "Content-Length": "0",
        "Host": "httpbin:8000",
        "User-Agent": "curl/7.35.0",
        "X-B3-Parentspanid": "e86c2481d7f2d835",
        "X-B3-Sampled": "1",
        "X-B3-Spanid": "49ea6ca1091c8648",
        "X-B3-Traceid": "beeed0f13b512317e86c2481d7f2d835"
    }
}
```

After sending the traffic above, both version of `HTTPbin` will see traffic logs because version-2 is being mirrored with version-1.

Note: Run two bash terminals for both `HTTPbin` versions to see the logs when the curl command from above is executed.

```console
[root@osc01 (istio-lab)istio-scripts]# k log -f httpbin-v2-67c5fc7ffb-9qr7k -c httpbin
log is DEPRECATED and will be removed in a future version. Use logs instead.
[2019-04-18 01:49:29 +0000] [1] [INFO] Starting gunicorn 19.9.0
[2019-04-18 01:49:29 +0000] [1] [INFO] Listening at: http://0.0.0.0:80 (1)
[2019-04-18 01:49:29 +0000] [1] [INFO] Using worker: sync
[2019-04-18 01:49:29 +0000] [8] [INFO] Booting worker with pid: 8
127.0.0.1 - - [18/Apr/2019:04:41:54 +0000] "GET /headers HTTP/1.1" 200 343 "-" "curl/7.35.0"
```

```console
[root@osc01 (istio-lab)istio-scripts]# k log -f httpbin-v1-6569dfb499-r7xhh -c httpbin
log is DEPRECATED and will be removed in a future version. Use logs instead.
[2019-04-18 01:49:29 +0000] [1] [INFO] Starting gunicorn 19.9.0
[2019-04-18 01:49:29 +0000] [1] [INFO] Listening at: http://0.0.0.0:80 (1)
[2019-04-18 01:49:29 +0000] [1] [INFO] Using worker: sync
[2019-04-18 01:49:29 +0000] [8] [INFO] Booting worker with pid: 8
127.0.0.1 - - [18/Apr/2019:04:41:54 +0000] "GET /headers HTTP/1.1" 200 303 "-" "curl/7.35.0"
```

Finally, to see the traffic internals of both `HTTPbin` versions, run the following command. But first, run the sleep service CURL command for HTTPbin in another console to see the internal logs

For the command below:

$SLEEP_POD is the sleep service pod name
10.1.230.202 is the IP address of `HTTPbin` version 1 pod
10.1.230.247 is the IP address of `HTTPbin` version 2 pod

```console
# kubectl exec -it $SLEEP_POD -c istio-proxy -- sudo tcpdump -A -s 0 host 10.1.230.202 or host 10.1.230.247
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
04:52:11.317838 IP sleep-7549f66447-rmcl8.51292 > 10-1-230-247.httpbin.istio-lab.svc.cluster.local.80: Flags [P.], seq 3682856920:3682857728, ack 2162198975, win 245, options [nop,nop,TS val 10822498 ecr 10205363], length 808: HTTP: GET /headers HTTP/1.1
E..\..@.@...
...
....\.P.............).....
..#b....GET /headers HTTP/1.1
host: httpbin-shadow:8000
user-agent: curl/7.35.0
accept: */*
x-forwarded-proto: http
x-request-id: daf32666-b70f-9fdb-8935-f451be45e582
x-envoy-decorator-operation: httpbin.istio-lab.svc.cluster.local:8000/*
x-istio-attributes: Cj8KF2Rlc3RpbmF0aW9uLnNlcnZpY2UudWlkEiQSImlzdGlvOi8vaXN0aW8tbGFiL3NlcnZpY2VzL2h0dHBiaW4KQQoYZGVzdGluYXRpb24uc2VydmljZS5ob3N0EiUSI2h0dHBiaW4uaXN0aW8tbGFiLnN2Yy5jbHVzdGVyLmxvY2FsCiUKGGRlc3RpbmF0aW9uLnNlcnZpY2UubmFtZRIJEgdodHRwYmluCiwKHWRlc3RpbmF0aW9uLnNlcnZpY2UubmFtZXNwYWNlEgsSCWlzdGlvLWxhYgo9Cgpzb3VyY2UudWlkEi8SLWt1YmVybmV0ZXM6Ly9zbGVlcC03NTQ5ZjY2NDQ3LXJtY2w4LmlzdGlvLWxhYg==
x-b3-traceid: d76c34ca9d2afafe2ca5c6f1ccf98983
x-b3-spanid: 2ca5c6f1ccf98983
x-b3-sampled: 1
x-envoy-internal: true
x-forwarded-for: 10.1.230.225
content-length: 0


04:52:11.322771 IP 10-1-230-247.httpbin.istio-lab.svc.cluster.local.80 > sleep-7549f66447-rmcl8.51292: Flags [P.], seq 1:580, ack 808, win 266, options [nop,nop,TS val 10822503 ecr 10822498], length 579: HTTP: HTTP/1.1 200 OK
E..w. @.?...
...
....P.\...........
.D.....
..#g..#bHTTP/1.1 200 OK
server: istio-envoy
date: Thu, 18 Apr 2019 04:52:11 GMT
content-type: application/json
content-length: 343
access-control-allow-origin: *
access-control-allow-credentials: true
x-envoy-upstream-service-time: 2

{
  "headers": {
    "Accept": "*/*",
    "Content-Length": "0",
    "Host": "httpbin-shadow:8000",
    "User-Agent": "curl/7.35.0",
    "X-B3-Parentspanid": "2ca5c6f1ccf98983",
    "X-B3-Sampled": "1",
    "X-B3-Spanid": "5068095531ea1c31",
    "X-B3-Traceid": "d76c34ca9d2afafe2ca5c6f1ccf98983",
    "X-Envoy-Internal": "true"
  }
}

04:52:11.322786 IP sleep-7549f66447-rmcl8.51292 > 10-1-230-247.httpbin.istio-lab.svc.cluster.local.80: Flags [.], ack 580, win 254, options [nop,nop,TS val 10822503 ecr 10822503], length 0
E..4..@.@..*
...
....\.P...................
..#g..#g
```

## Control Ingress Traffic

Within Kubernetes, K8s ingress resource specific services that should be exposed outside the cluster. Within istio, use the Gateway to allow monitoring and applying route rules to traffic entering the cluster.

In this exercise, Istio will be configured to expose a service outside of Istio using Gateway.

Note: Make sure you are `cd` into the `istio` directory.

Identify the ingress gateway details, and take a note of the external IP address:

```console
# kgsvc istio-ingressgateway
NAME                   TYPE           CLUSTER-IP   EXTERNAL-IP       PORT(S)                                                                                                                                      AGE
istio-ingressgateway   LoadBalancer   10.0.0.156   192.168.142.250   80:31380/TCP,443:31390/TCP,31400:31400/TCP,15029:31391/TCP,15030:31040/TCP,15031:31396/TCP,15032:30231/TCP,15443:32612/TCP,15020:31795/TCP   3d
```

The external-ip will be the ingress gateway and since it exist, this confirms an external loadbalancer is deployed, which is Keepalived. If the external-ip is `none` or `pending`, then the existing environment doesn't have an external load balancer for the ingress gateway. To bypass, the service type can be changed from `ClusterIP` to `NodePort`

Capture the external-ip host, port and secure ingress port.

Identify the Ingress host, which is `192.168.142.150`:
```console
# kgsvc istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
192.168.142.250
```

Identify the Ingress port, which is `80`"
```console
# kgsvc istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}'
80
```

Identify the Secure Ingress Port, which is `443`:
```console
# kgsvc istio-ingressgateway -o jsonpath=.spec.ports[?(@.name=="https")].port}'
443
```
If DNS is enabled or if a hostname is defined, the external-ip value might be a host name, instead of an IP address. 

If an external load balancer is not enabled, leverage the service nodeport to identify the ingress port and secure port.

For ingress port without an external load balancer, which is `31380`:
```console
# kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}'
31380
```

For secure ingress port with an external load balancer, which is `31390`:
```console
# kubectl -n istio-system get service -ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}'
31390
```

### Configure ingress using Istio Gateway

The ingress gateway will operate at the edge to receive incoming HTTP/TCP connections. It configures exposed ports, protocols etc. Outside of kubernetes ingress resources, this gateway will not contain any traffic routing configurations. 

To enable such routing configurations for ingress traffic, istio routing rules will be enabled. This is similar to internal service requests.

First, create an Istio gateway on port 80 for HTTP traffic:

```console
# kubectl apply -f 23-create-gateway-httpbin80.yaml -n istio-lab
gateway.networking.istio.io/httpbin-gateway created
```

Configure and enable routes for traffic entering the gateway for `Httpbin` on Port 8000, for two route rules under paths `/status` and `/delay`.

```console
# kubectl apply -f 24-apply-routes-httpbin80.yaml -n istio-lab
virtualservice.networking.istio.io/httpbin configured
```

Access the `HTTPbin` service using curl, confirm and correct ingress host and ingress port:

```console
# curl -I -HHost:httpbin.example.com h/192.168.142.250:80/status/200
HTTP/1.1 200 OK
server: istio-envoy
date: Thu, 18 Apr 2019 05:37:15 GMT
content-type: text/html; charset=utf-8
access-control-allow-origin: *
access-control-allow-credentials: true
content-length: 0
x-envoy-upstream-service-time: 3
```

Note the `access-control-allow-origin` value is `*`. What this means is that the incoming request is coming for any possible host.

Try and test out the same curl command above, but with a different host name. The output should be a `HTTP 404 error`. This is becuase the current ingress external-ip and/or hostname is the only istio configured gateway that can accept incoming traffic.

```console
# curl -I -HHost:httpbin.example1.com http://192.168.142.250:80/status/200

HTTP/1.1 404 Not Found
date: Thu, 18 Apr 2019 05:47:04 GMT
server: istio-envoy
transfer-encoding: chunked
```

### Access ingress services using a browser

To resolve `HTTPbin`'s current host name of `httpbin.example.com`, configure the host with DNS server. Since we're not working with a DNS environment, use `*` in place of host. This allows gateway and virtual configurations to allow traffic coming from any host value. 

Apply the new ingress gateway and virtual service YMAL to allow incoming traffic through `HTTPbin` via any open `*` host:

```console
# kubectl apply -f 25-apply-httpbin-vs-dr-anyhost.yaml -n istio-lab
gateway.networking.istio.io/httpbin-gateway configured
virtualservice.networking.istio.io/httpbin configured
```

Navigate to a browser and enter: `http://192.168.142.250:31380/headers`

Which is the `http://INGRESS_HOST:INGRESS_PORT_WO_LOADBALANCER`

Output should look like this:
```console
{
  "headers": {
    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3", 
    "Accept-Encoding": "gzip, deflate", 
    "Accept-Language": "en-US,en;q=0.9", 
    "Content-Length": "0", 
    "Host": "192.168.142.250:31380", 
    "Upgrade-Insecure-Requests": "1", 
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.103 Safari/537.36", 
    "X-B3-Parentspanid": "ad30f94b17ed48c3", 
    "X-B3-Sampled": "1", 
    "X-B3-Spanid": "3bc6f2a41d96c60f", 
    "X-B3-Traceid": "5dd2dc4ea1db35ccad30f94b17ed48c3", 
    "X-Envoy-Internal": "true"
  }
}
```

To re-cap, the Istio gateway configuration allows external traffic to enter based on traffic and policy features configured at the edge services.

## Control Egress Traffic

Any outbound traffic from a pod is re-directed to its sidecar proxy. Accessbility of URLs outside of the cluster depends on proxy configuration. 

By default, Istio configures sidecar proxy to pass through requests of unknown services. 

In this exercise, egress will enable stricter control by: 
-Allow sidecar to pass requests through to services that aren't configured inside the mesh
-Configure service entries to provide controlled access to external services
-Bypass sidecar for range of IPs.

### Envoy passthrough to external services

Allow outbound traffic for `global.outboundTrafficPolicy.mode` that allows/restricts sidecar to handle external services. This is specifically for services that are not defined by Istio's service registry. The property will have options of `ALLOW_ANY` and `REGISTRY_ONLY`:

-`ALLOW_ANY` is default and allows calls to unknown services to pass through
-`REGISTER_ONLY` blocks any host that doesn't have a HTTP service or service entry defined within Istio.

We'll start with validating the existing value of istio config map and it's outbound traffic policy mode:

```console
[root@osc01 (istio-system)~]# kubectl get configmap istio -n istio-system -o yaml | grep -o "mode: ALLOW_ANY"
mode: ALLOW_ANY
```

Since it allows all calls, run the sleep pod to confirm outgoing calls to hosts such as google, cnn etc.:

```console
[root@osc01 (istio-lab)~]# kubectl exec -it sleep-7549f66447-rmcl8 -c sleep -- curl -I https://www.google.com | grep  "HTTP/"; kubectl exec -it sleep-7549f66447-rmcl8 -c sleep -- curl -I https://edition.cnn.com | grep "HTTP/"
HTTP/1.1 200 OK
HTTP/1.1 200 OK
```
This confirms that with egress outbound policy, all traffic is permissible through Istio. 
This drawback with this setup, any Istio calls to external services will not be available in the Mixer log. This can be changed by enabling controlled access for HTTP headers.

### Controlled access to external services

The `ServiceEntry` configruations allows access to publicly accessible services within Istio cluster. We'll enable access to an HTTP & HTTPS external service without losing monitoring and control features.

Enable access to external service by changing `global.outboundTrafficPolicy.mode` from `ALLOW_ANY` to `REGISTRY_ONLY` within istio config map under `istio-system` namespace.

```console
[root@osc01 (istio-system)~]# kubectl get configmap istio -n istio-system -o yaml | sed 's/mode: ALLOW_ANY/mode: REGISTRY_ONLY/g' | kubectl replace -n istio-system -f -
configmap/istio replaced
```

Make requests to the external services like: Google, CNN using sleep pod within `istio-lab` napesapce and validate the request is blocked. 
Reason why its blocked is because a `ServiceEntry` for these external services hasn't been created with the new traffic policy mode.

Note: it might take a couple of changes for the egress outbound policy changes to take affect. Re-run the command, if the connection is a success.

```console
[root@osc01 (istio-lab)~]# kubectl exec -it sleep-7549f66447-rmcl8 -c sleep -- curl -I https://www.google.com | grep  "HTTP/"; kubectl exec -it sleep-7549f66447-rmcl8 -c sleep -- curl -I https://edition.cnn.com | grep "HTTP/"
command terminated with exit code 35
command terminated with exit code 35
```
### Accessing external HTTP service

Create a `ServiceEntry` YAML to allow HTTP access to external `HTTPbin` service.

```console
[root@osc01 (istio-lab)istio-scripts]# kubectl apply -f 26-create-serviceentry-http-ext.yaml -n istio-lab
serviceentry.networking.istio.io/httpbin-ext created
```

Make a request to the external `HTTPbin` service from sleep pod:

```console
[root@osc01 (istio-lab)istio-scripts]# kubectl exec -it sleep-7549f66447-rmcl8 -c sleep -- curl http://httpbin.org/headers
{
  "headers": {
    "Accept": "*/*",
    "Host": "httpbin.org",
    "User-Agent": "curl/7.35.0",
    "X-B3-Sampled": "1",
    "X-B3-Spanid": "1f6269a5421a30ca",
    "X-B3-Traceid": "d9c73798e2a6b0711f6269a5421a30ca",
    "X-Envoy-Decorator-Operation": "httpbin.org:80/*",
    "X-Istio-Attributes": "CiwKHWRlc3RpbmF0aW9uLnNlcnZpY2UubmFtZXNwYWNlEgsSCWlzdGlvLWxhYgopChhkZXN0aW5hdGlvbi5zZXJ2aWNlLm5hbWUSDRILaHR0cGJpbi5vcmcKPQoKc291cmNlLnVpZBIvEi1rdWJlcm5ldGVzOi8vc2xlZXAtNzU0OWY2NjQ0Ny1ybWNsOC5pc3Rpby1sYWIKKQoYZGVzdGluYXRpb24uc2VydmljZS5ob3N0Eg0SC2h0dHBiaW4ub3Jn"
  }
}
```

The HTTP headers are added by the sidecar and defined in the `X-Envoy-Decorator-Operation` property.

Check the logs of the istio sidecar for sleep pod and validate it's available.

!DIDN'T WORK!
```console
kubectl logs $SOURCE_POD -c istio-proxy | tail
Output -> check logsoutput.txt under path 
```

Check the Mixer log to see if Istio is deployed in `istio-system`.

```console
[root@osc01 (istio-lab)istio-scripts]# kubectl -n istio-system logs -l istio-mixer-type=telemetry -c mixer | grep 'httpbin.org'
{"level":"info","time":"2019-04-18T21:06:16.043445Z","instance":"accesslog.logentry.istio-system","apiClaims":"","apiKey":"","clientTraceId":"","connection_security_policy":"unknown","destinationApp":"","destinationIp":"AAAAAAAAAAAAAP//AAAAAA==","destinationName":"unknown","destinationNamespace":"default","destinationOwner":"unknown","destinationPrincipal":"","destinationServiceHost":"","destinationWorkload":"unknown","grpcMessage":"","grpcStatus":"","httpAuthority":"httpbin.org","latency":"761.837µs","method":"GET","permissiveResponseCode":"none","permissiveResponsePolicyID":"none","protocol":"http","receivedBytes":143,"referer":"","reporter":"source","requestId":"131c6767-1354-9b24-9859-528b6e84b583","requestSize":0,"requestedServerName":"","responseCode":404,"responseFlags":"NR","responseSize":0,"responseTimestamp":"2019-04-18T21:06:16.043896Z","sentBytes":54,"sourceApp":"sleep","sourceIp":"AAAAAAAAAAAAAP//CgHm5Q==","sourceName":"sleep-7549f66447-rmcl8","sourceNamespace":"istio-lab","sourceOwner":"kubernetes://apis/apps/v1/namespaces/istio-lab/deployments/sleep","sourcePrincipal":"","sourceWorkload":"sleep","url":"/headers","userAgent":"curl/7.35.0","xForwardedFor":"0.0.0.0"}
{"level":"info","time":"2019-04-18T21:06:30.896517Z","instance":"accesslog.logentry.istio-system","apiClaims":"","apiKey":"","clientTraceId":"","connection_security_policy":"unknown","destinationApp":"","destinationIp":"AAAAAAAAAAAAAP//AAAAAA==","destinationName":"unknown","destinationNamespace":"default","destinationOwner":"unknown","destinationPrincipal":"","destinationServiceHost":"","destinationWorkload":"unknown","grpcMessage":"","grpcStatus":"","httpAuthority":"httpbin.org","latency":"168.903µs","method":"GET","permissiveResponseCode":"none","permissiveResponsePolicyID":"none","protocol":"http","receivedBytes":143,"referer":"","reporter":"source","requestId":"eecb6335-4448-9074-9728-f2616fc7081f","requestSize":0,"requestedServerName":"","responseCode":404,"responseFlags":"NR","responseSize":0,"responseTimestamp":"2019-04-18T21:06:30.896607Z","sentBytes":54,"sourceApp":"sleep","sourceIp":"AAAAAAAAAAAAAP//CgHm5Q==","sourceName":"sleep-7549f66447-rmcl8","sourceNamespace":"istio-lab","sourceOwner":"kubernetes://apis/apps/v1/namespaces/istio-lab/deployments/sleep","sourcePrincipal":"","sourceWorkload":"sleep","url":"/headers","userAgent":"curl/7.35.0","xForwardedFor":"0.0.0.0"}
{"level":"info","time":"2019-04-18T21:25:36.896593Z","instance":"accesslog.logentry.istio-system","apiClaims":"","apiKey":"","clientTraceId":"","connection_security_policy":"unknown","destinationApp":"","destinationIp":"A1WakA==","destinationName":"unknown","destinationNamespace":"default","destinationOwner":"unknown","destinationPrincipal":"","destinationServiceHost":"httpbin.org","destinationWorkload":"unknown","grpcMessage":"","grpcStatus":"","httpAuthority":"httpbin.org","latency":"149.492349ms","method":"GET","permissiveResponseCode":"none","permissiveResponsePolicyID":"none","protocol":"http","receivedBytes":186,"referer":"","reporter":"source","requestId":"cd638beb-58e1-9357-b721-5db002e470a9","requestSize":0,"requestedServerName":"","responseCode":200,"responseFlags":"-","responseSize":575,"responseTimestamp":"2019-04-18T21:25:37.045863Z","sentBytes":888,"sourceApp":"sleep","sourceIp":"AAAAAAAAAAAAAP//CgHm5Q==","sourceName":"sleep-7549f66447-rmcl8","sourceNamespace":"istio-lab","sourceOwner":"kubernetes://apis/apps/v1/namespaces/istio-lab/deployments/sleep","sourcePrincipal":"","sourceWorkload":"sleep","url":"/headers","userAgent":"curl/7.35.0","xForwardedFor":"0.0.0.0"}
```

Within the Mixer log, the `destinationServiceHost` attribute is set to `httpbin.org`. Using egress traffic control, monitoring is enabled for external HTTP services including related information around access.

### Accessing external HTTPS service

Create a `ServiceEntry` YAML to allow HTTPS access to external `HTTPbin` service.

```console
[root@osc01 (istio-lab)istio-scripts]# kubectl apply -f 27-create-serviceentry-https-ext.yaml -n istio-lab
serviceentry.networking.istio.io/google created
```

Initiate a request to the external HTTPS service to google from the sleep pod:

```console
[root@osc01 (istio-lab)istio-scripts]# kubectl exec -it sleep-7549f66447-rmcl8 -c sleep -- curl -I https://www.google.com | grep  "HTTP/"
HTTP/1.1 200 OK
```

Check the sidecar proxy log:

!DIDN'T WORK!
```console
kubectl logs $SOURCE_POD -c istio-proxy | tail
Output -> check logsoutput.txt under path 
```
Check the mixer log to see if Istio is deployed in `istio-system`.

```console
[root@osc01 (istio-lab)~]# kubectl -n istio-system logs -l istio-mixer-type=telemetry -c mixer | grep 'www.google.com'
{"level":"info","time":"2019-04-20T02:51:39.539649Z","instance":"tcpaccesslog.logentry.istio-system","connectionDuration":"0s","connectionEvent":"open","connection_security_policy":"unknown","destinationApp":"","destinationIp":"rNmkhA==","destinationName":"unknown","destinationNamespace":"default","destinationOwner":"unknown","destinationPrincipal":"","destinationServiceHost":"www.google.com","destinationWorkload":"unknown","protocol":"tcp","receivedBytes":0,"reporter":"source","requestedServerName":"www.google.com","responseFlags":"","sentBytes":0,"sourceApp":"sleep","sourceIp":"CgHnFg==","sourceName":"sleep-7549f66447-rmcl8","sourceNamespace":"istio-lab","sourceOwner":"kubernetes://apis/apps/v1/namespaces/istio-lab/deployments/sleep","sourcePrincipal":"","sourceWorkload":"sleep","totalReceivedBytes":0,"totalSentBytes":0}
{"level":"info","time":"2019-04-20T02:51:39.635535Z","instance":"tcpaccesslog.logentry.istio-system","connectionDuration":"131.633351ms","connectionEvent":"close","connection_security_policy":"unknown","destinationApp":"","destinationIp":"rNmkhA==","destinationName":"unknown","destinationNamespace":"default","destinationOwner":"unknown","destinationPrincipal":"","destinationServiceHost":"www.google.com","destinationWorkload":"unknown","protocol":"tcp","receivedBytes":561,"reporter":"source","requestedServerName":"www.google.com","responseFlags":"","sentBytes":3225,"sourceApp":"sleep","sourceIp":"CgHnFg==","sourceName":"sleep-7549f66447-rmcl8","sourceNamespace":"istio-lab","sourceOwner":"kubernetes://apis/apps/v1/namespaces/istio-lab/deployments/sleep","sourcePrincipal":"","sourceWorkload":"sleep","totalReceivedBytes":561,"totalSentBytes":3225}
```

### Manage traffic to external services

Similar to inter-cluster requests, Istio routing rules can also be set for external services that are accessed using `ServiceEntry` configurations. 

Set timeout rules on class to the `httpbin.org` service using sleep pod and initiating a `delay`.

From inside the pod, initiate a curl request to delay endpoint of `httpbin.org` external service

```console
[root@osc01 (istio-lab)~]# kubectl exec -it  sleep-7549f66447-rmcl8 -c sleep sh
# curl -o /dev/null -s -w "%{http_code}\n" http://httpbin.org/delay/5
200
```

Within 5 seconds, the output should be 200 (OK). Exit out of the pod and use `kubectl` to apply a 3s timeout on calls to the `httpbin.org` external service.

```console
[root@osc01 (istio-lab)istio-scripts]# kubectl apply -f 28-apply-3sec-timeout-httpbin.yaml
virtualservice.networking.istio.io/httpbin-ext created
```

After a few seconds, run the same curl request:

```console
[root@osc01 (istio-lab)istio-scripts]# kubectl exec -it  sleep-7549f66447-rmcl8 -c sleep sh
# curl -o /dev/null -s -w "%{http_code}\n" http://httpbin.org/delay/5
504
```

This time, there's 504 (gateway timeout) error versus a 200 (OK) output. Although `httpbin.org` was waiting 5 seconds, istio's timeout request set at 3 seconds, cut off the request.

### Direct access to external services

Bypassing Istio for a specific IP range can be setup, by configuring sidecars to prevent them from intercepting external requests. To setup this bypass change `global.proxy.includeIPRanges` or `global.proxy.excludeIPRanges` configuration option and update `istio-sidecar-injector` configuration. This process will affect all future pod deployments. 

This approach is only recommended as a last resort when, for performance or other reasons, external access can't be configured using sidecar as Istio's features will be completely disabled for the specified IPs. 

An easy way to exclude all external IPs from being redirected to the sidecar is to set `global.proxy.includeIPRanges` for IP range or ranges used for internal cluster services.  

#### Determine the IP ranges

Set the value for `global.proxy.includeIPRanges` in accordance to the cluster provider.

For IBM Cloud Private, CD to the cluster folder within ICP.

```console
[root@osc01 (istio-lab)icp3.1.2]# pwd
/opt/ibm/icp3.1.2
[root@osc01 (istio-lab)icp3.1.2]# cat cluster/config.yaml | grep service_cluster_ip_range
# service_cluster_ip_range: 10.0.0.0/16
service_cluster_ip_range: 10.0.0.1/24
```

#### Configure proxy bypass

Update `istio-sidecar-injector` configmap using the IP ranges specific to your cluster provider. 

```console
# helm template install/kubernetes/helm/istio --set global.proxy.includeIPRanges="10.0.0.1/24" -x templates/sidecar-injector-configmap.yaml | kubectl apply -f -
configmap/istio-sidecar-injector created
```

#### Access external services

Now that the bypass config will affect new deployments, the sleep pod will need to be redeployed.

```console
# kubectl apply -f 29-create-sleep-service-account.yaml
serviceaccount/sleep created
service/sleep created
deployment.extensions/sleep configured
```

After `istio-sidecar-injector` configmap and redeployment of the `sleep` pod, Istio's sidecar will intercept and manage internal requests within the cluster. Any external requests will bypass the sidecar and go to its destination. 

```console
[root@osc01 (istio-lab)istio-scripts]# k exec -it sleep-75b799df5f-csgb6 -c sleep curl http://httpbin.org/headers
{
  "headers": {
    "Accept": "*/*",
    "Host": "httpbin.org",
    "User-Agent": "curl/7.35.0",
    "X-B3-Sampled": "1",
    "X-B3-Spanid": "986b188d5f866f39",
    "X-B3-Traceid": "8ab07b42ddc84025986b188d5f866f39",
    "X-Envoy-Decorator-Operation": "httpbin.org:80/*",
    "X-Envoy-Expected-Rq-Timeout-Ms": "3000",
    "X-Istio-Attributes": "Cj0KCnNvdXJjZS51aWQSLxIta3ViZXJuZXRlczovL3NsZWVwLTc1Yjc5OWRmNWYtY3NnYjYuaXN0aW8tbGFiCikKGGRlc3RpbmF0aW9uLnNlcnZpY2UuaG9zdBINEgtodHRwYmluLm9yZwopChhkZXN0aW5hdGlvbi5zZXJ2aWNlLm5hbWUSDRILaHR0cGJpbi5vcmcKLAodZGVzdGluYXRpb24uc2VydmljZS5uYW1lc3BhY2USCxIJaXN0aW8tbGFi"
  }
}
```

Notice the `headers` property, there aren't any for either HTTP or HTTPS external services. Logs will also not appear in the mixer log. Bypassing the istio sidecars means you can no longer monitor the access to external services.

#### Summary

In this section, we looked at:

1) Configure sidecar to allow access for any external service:

This directs traffic through Istio sidecar, include calls to external services that unknown within the mesh. There is no monitoring of the external services, so traffic control can't be fully leveraged. 

2) Use service entry to register an accessible external service inside the mesh. (Recommended)

Creating service entries allows use of all the same Istio features for calls to services inside and outside of the cluster. Monitoring is always available to external services by setting timeouts to external service. 

3) Configure Istio sidecar to exclude external IP that are not defined within the internal mesh range.

This bypasses sidecar and gives access to the external services directly. To do this setup, requires access to the cluster manager and knowledge of the environment where Istio is hosted. Monitoring will not be available for access of external services and istio features will not be available for traffic management.

Note: Implementing egress traffic controle more securly, direct egress traffic through an egress gateway and review security concerns.

# Policies

## Enabling Policy Enforcement

This shows how to enable istio policy enforcement

The default Istio install has policy enforcemnt diabled. To install Istio with policy enforcement, use `--set global.disablePolicyChecks=false`

Check the status of policy enforcement for Istio

```console
[root@osc01 (istio-lab)policies]# kubectl -n istio-system get cm istio -o jsonpath="{@.data.mesh}" | grep disablePolicyChecks
disablePolicyChecks: false
```

Since policy enforcement is enabled, no further action is required. 

If it is disabled, edit the `istio` config map and enable policy checks.

Note: validate you're in the following path to confirm the status of the policy check:


```console
# pwd
/opt/ibm/icp3.1.2/cluster/istio-1.1.3

# helm template install/kubernetes/helm/istio --namespace=istio-system -x templates/configmap.yaml --set global.disablePolicyChecks=false | kubectl -n istio-system replace -f -
configmap/istio replaced
```
## Enabling Rate Limits

This shows how to use Istio by dynamically limiting traffic to a service. 

Validate the BookInfo app is installed and its virtualservice is deployed. 

```console
# kubectl apply -f 03-create-reviews-virtual-service.yaml
virtualservice.networking.istio.io/productpage unchanged
virtualservice.networking.istio.io/reviews unchanged
virtualservice.networking.istio.io/ratings unchanged
virtualservice.networking.istio.io/details unchanged
```

Configure Istio to rate limit traffic to the `productpage` based on IP address of the client. Use `X-Forwarded-For ` request header as the client IP address. Use conditional rate limit that exempts logged in users. 

Enable a memory quote `memquota` to enable rate limits. If running in production, `redis` is required and its `redisquota` quota. 

Both `memquota` and `redisquota` support the quota template, so configuration to enable rate limits on both adapters is the same. 

Client side rate limitions is in 2 parts:
-`QuotaSpec` defines quota name and amount that the client should request.
-`QuotaSpecBinding` conditionally associates `QuotaSpec` with one or more services.

Mixer side rate limitations:
-`quota instance` defines quota by Mixer
-`memquota handler` defines `memquota` configuration
-`quota rule` defines when quota instance is dispatched to `memquota` adapter.

Run the `memquota` YAML to enable rate limits:

Note: Ensure you're running Istio v1.1.3 and higher.

```console
[root@osc01 (istio-system)policies]# kubectl apply -f 01-create-memquota-ratelimit.yaml
handler.config.istio.io/quotahandler created
instance.config.istio.io/requestcountquota created
quotaspec.config.istio.io/request-count created
quotaspecbinding.config.istio.io/request-count created
rule.config.istio.io/quota created
```

The memquota handler has 3 different rate limits. By default, if there are no overrides, there are 500 requests per one second (1s). 
From the YAML above, two overrides are defined:

1) 1 request every 5 seconds, if destination is `reviews`.
2) 2 requests every 5 seconds, if destination is `productpage`.

When a request is processed, first matching override is picked.

If working with a production environment, the `redisquota` handles 4 different rate limit schemes. it uses `ROLLING_WINDOW` algorithm for quota check and defines `bucketDuration` of 500s. By default, if there are no overrides, there are 500 requests per one second (1s). 
It has three overrides defined: 

1) 1 request, if destination is `reviews`
2) Second is 500, if destination is `productpage` and source is `10.28.11.20`
3) 2, if destination is `productpage

When a request is processed, first matching override is picked.

Validate the `quota instance` was created:

```YAML
# kubectl -n istio-system get instance requestcountquota -o yaml
apiVersion: config.istio.io/v1alpha2
kind: instance
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"config.istio.io/v1alpha2","kind":"instance","metadata":{"annotations":{},"name":"requestcountquota","namespace":"istio-system"},"spec":{"compiledTemplate":"quota","params":{"dimensions":{"destination":"destination.labels[\"app\"] | destination.service.name | \"unknown\"","destinationVersion":"destination.labels[\"version\"] | \"unknown\"","source":"request.headers[\"x-forwarded-for\"] | \"unknown\""}}}}
  creationTimestamp: 2019-04-20T04:37:31Z
  generation: 1
  name: requestcountquota
  namespace: istio-system
  resourceVersion: "177417"
  selfLink: /apis/config.istio.io/v1alpha2/namespaces/istio-system/instances/requestcountquota
  uid: 02be487a-6326-11e9-8154-00505632f6a0
spec:
  compiledTemplate: quota
  params:
    dimensions:
      destination: destination.labels["app"] | destination.service.name | "unknown"
      destinationVersion: destination.labels["version"] | "unknown"
      source: request.headers["x-forwarded-for"] | "unknown"
```

The quota template has 3 dimensions used by `memquota` (as shown above) or in `redisquota` to enable overrides on requests that match certain attributes. 

Confirm the `quota rule` was created:

```yaml
# kubectl -n istio-system get rule quota -o yaml
apiVersion: config.istio.io/v1alpha2
kind: rule
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"config.istio.io/v1alpha2","kind":"rule","metadata":{"annotations":{},"name":"quota","namespace":"istio-system"},"spec":{"actions":[{"handler":"quotahandler","instances":["requestcountquota"]}]}}
  creationTimestamp: 2019-04-20T04:37:31Z
  generation: 1
  name: quota
  namespace: istio-system
  resourceVersion: "177421"
  selfLink: /apis/config.istio.io/v1alpha2/namespaces/istio-system/rules/quota
  uid: 02c309b0-6326-11e9-8154-00505632f6a0
spec:
  actions:
  - handler: quotahandler
    instances:
    - requestcountquota
```

A rule tells Mixer to invoke either `memquota` or `redisquota` handler and pass it to the `requestcountquota` instance. This maps `quota` template to `memquota` or `redisquota` handler.

Confirm the `QuotaSpec` was created:

```yaml
# kubectl -n istio-system get QuotaSpec request-count -o yaml
apiVersion: config.istio.io/v1alpha2
kind: QuotaSpec
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"config.istio.io/v1alpha2","kind":"QuotaSpec","metadata":{"annotations":{},"name":"request-count","namespace":"istio-system"},"spec":{"rules":[{"quotas":[{"charge":1,"quota":"requestcountquota"}]}]}}
  creationTimestamp: 2019-04-20T04:37:31Z
  generation: 1
  name: request-count
  namespace: istio-system
  resourceVersion: "177418"
  selfLink: /apis/config.istio.io/v1alpha2/namespaces/istio-system/quotaspecs/request-count
  uid: 02bf6d7d-6326-11e9-8154-00505632f6a0
spec:
  rules:
  - quotas:
    - charge: 1
      quota: requestcountquota
```

The `QuotaSpec` translates to 1 for `requestcountquota`.

Confirm the `QuotaSpecBinding` was created:

```yaml
# kubectl -n istio-system get QuotaSpecBinding request-count -o yaml
apiVersion: config.istio.io/v1alpha2
kind: QuotaSpecBinding
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"config.istio.io/v1alpha2","kind":"QuotaSpecBinding","metadata":{"annotations":{},"name":"request-count","namespace":"istio-system"},"spec":{"quotaSpecs":[{"name":"request-count","namespace":"istio-system"}],"services":[{"name":"productpage","namespace":"default"}]}}
  creationTimestamp: 2019-04-20T04:37:31Z
  generation: 1
  name: request-count
  namespace: istio-system
  resourceVersion: "177419"
  selfLink: /apis/config.istio.io/v1alpha2/namespaces/istio-system/quotaspecbindings/request-count
  uid: 02c12497-6326-11e9-8154-00505632f6a0
spec:
  quotaSpecs:
  - name: request-count
    namespace: istio-system
  services:
  - name: productpage
    namespace: default
```

`QuotaSpecBinding` binds the `QuotaSpec` that was created to the services, in this case the `productpage` where its bound to the `request-count`. 

### Conditional Rate Limits

In the example above, rate limits are set for `productpage` at 2 requests per second / client IP. 

Now, we will explore exempt clients from rate limit if a user is logged in. 

Refresh the productpage, logged in as `jason` and the productpage is repeatedly refreshed, nothing should change because rate limit is not applied.

!DIDN'T WORK
After you log out, verify the rate limit does apply and you should see `RESOURCE_EXHAUSTED:Quota is exhausted for: requestcount`

### Understanding Rate Limits

Mixer applies rate limits to requests that match certain conditions. Every quota represents set of counters. The set is defined by quota dementions. If the # of requests in `expiration` exceed `maxAmount` Mixer will return a `RESOURCE_EXHAUSTED` message to sidecar. Sidecar returns a `HTTP 429` to the caller.

`memquota` / `redisquota` adapter enforces rate limits. 
`maxAmount` are configuration sets for default limits for all counters associated with a quota.

The default limit is applied, if the quota override doesn't match the request. The rate limit adapters (`memquota`/`redisquota`) selects first override that matches a request. Override does not specify all quota dimensions. 

Such rate limit policies can be enforced for any namespace instead of the entire istio framework. 

## Control Headers and Routing

Before getting started with headers and routing traffic, validate ingress is configured using a gateway.

First, customize the `httpbin` virtual service to enable two route rules for `/headers` and `/status`

```bash
[root@osc01 (istio-lab)policies]# pwd
/opt/ibm/icp3.1.2/cluster/istio-scripts/policies
[root@osc01 (istio-lab)policies]# kubectl apply -f 02-create-httpbin-gw-routerules.yaml
virtualservice.networking.istio.io/httpbin configured
```

### Output-producing adapaters

Using a policy adapter called `keyval`, that will return an output with a single field called `value`. Adapter is configured with a lookup table, which it uses to populate the output value. If there isn't an output, `NOT_FOUND` error status is shown if there isn't an instance key within the lookup table. 


First, deploy the `keyval` adapter. 

NOTE: Make sure the Image policies on ICP are updated to allow `gcr.io` install of `keyval`. This can be done through the ICP home page by clicking on hamburger icon and navigate to: Manage -> Resource Security -> Image Policies and create a new image policy.

```bash
[root@osc01 (istio-system)policies]# kubectl run keyval --image=gcr.io/istio-testing/keyval:release-1.1 --namespace istio-system --port 9070 --expose
kubectl run --generator=deployment/apps.v1beta1 is DEPRECATED and will be removed in a future version. Use kubectl create instead.
service/keyval created
deployment.apps/keyval created
```

Validate the keyval pod was created:
```bash
[root@osc01 (istio-system)policies]# kgp | grep keyval
keyval-7489bf9678-tm9qz                                 1/1     Running     0          77s
```

Create the keyval adapter by deploying its template and configuration descriptors:

```bash
[root@osc01 (istio-system)policies]# kubectl apply -f 03-create-keyval-template.yaml
template.config.istio.io/keyval created
[root@osc01 (istio-system)policies]# kubectl apply -f 04-create-keyval-adapter.yaml
adapter.config.istio.io/keyval created
```

Create a handler for the `keyval` demo adapter with a fixed lookup table

```bash
[root@osc01 (istio-system)policies]# kubectl apply -f 05-create-keyval-lookup-table.yaml
handler.config.istio.io/keyval created
```

Create an instance for the `keyval` handler with a `user` request header as a lookup key

```bash
[root@osc01 (istio-system)policies]# kubectl apply -f 06-create-keyval-instance.yaml
instance.config.istio.io/keyval created
```

### Request header operations

Confirm the external ingress host name & port name and validate the `httpbin` service is accessible through the ingress gateway. 

```bash
[root@osc01 (istio-system)policies]# kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
192.168.142.250
[root@osc01 (istio-system)policies]# kubectl -n istio-system get service istio-ingressgateway spec.ports[?(@.name=="http2")].port}'
80
[root@osc01 (istio-system)policies]# curl http://192.168.142.250:80/headers
{
  "headers": {
    "Accept": "*/*",
    "Content-Length": "0",
    "Host": "192.168.142.250",
    "User-Agent": "curl/7.29.0",
    "X-B3-Parentspanid": "8cff45448d3acdf8",
    "X-B3-Sampled": "1",
    "X-B3-Spanid": "1a8d07e1f78316b0",
    "X-B3-Traceid": "0851827ad17cf6bf8cff45448d3acdf8",
    "X-Envoy-Internal": "true"
  }
}
```

The output should be request headers as they are being received by `httpbin` service. 

Create a rule for the `keyval` demo adapter.

```bash
[root@osc01 (istio-system)policies]# kubectl apply -f 07-create-keyval-rule.yaml
rule.config.istio.io/keyval created
```

Next, issue a request to the ingress gateway with the header as `jason` as a key value. 

```bash
[root@osc01 (istio-system)policies]# curl -Huser:jason http://192.168.142.250:80/headers                     {
  "headers": {
    "Accept": "*/*",
    "Content-Length": "0",
    "Host": "192.168.142.250",
    "User": "jason",
    "User-Agent": "curl/7.29.0",
    "User-Group": "admin",
    "X-B3-Parentspanid": "072070077118f836",
    "X-B3-Sampled": "1",
    "X-B3-Spanid": "b930792eba15d803",
    "X-B3-Traceid": "3aaad922482642bf072070077118f836",
    "X-Envoy-Internal": "true"
  }
}
```

The `user-group` header is derived from the `keyval` rule application of the adapter. Within the rule, the expression `x.output.value` is returned from the `key-val` adapter. 

Update the `keyval` rule with URI path to a different virtual service route. 

```bash
[root@osc01 (istio-system)policies]# kubectl apply -f 08-update-keyval-rule-newURI.yaml
rule.config.istio.io/keyval configured
```

Re-run the request to ingress gateway for user `jason`.

```bash
[root@osc01 (istio-system)policies]# curl -Huser:jason -I http://192.168.142.250:80/headers
HTTP/1.1 418 Unknown
server: istio-envoy
date: Mon, 22 Apr 2019 03:34:36 GMT
x-more-info: http://tools.ietf.org/html/rfc2324
access-control-allow-origin: *
access-control-allow-credentials: true
content-length: 135
x-envoy-upstream-service-time: 7
```

Note that the ingress gateway changed the route AFTER the rule is applied to the `keyval` adapter. 

Note: Modified requests based on microservice rules are not checked against the policy engine within the same proxy. A best practice is to use this feature in gateways so server-side policy checks aren't impacted.

## Denials and White/Black Listing

Control access to a service using denials, and attribute based white/black listing based on IP address.

Before getting started, validate the `BookInfo` app is deployed and initialize the `reviews` microservice for user `jason` to version-2 and all requests from any user to version-3. 

```bash
[root@osc01 (istio-lab)policies]# kubectl apply -f 09-update-bookinfo-vs-foruser.yaml
virtualservice.networking.istio.io/reviews configured
```

### Simple Denials

Access to a microservice can be based on any attributes that are available within Mixer. This access control is based on denying requests using Mixer selectors.

Login to the Bookinfo App as user `jason`, you will only see reviews-v2 (Black stars) no matter how many times you refresh the page. The ratings service is being called by reviews-v2 service.

If you change to another user, or logout, the reviews will only be called to reviews-v3 (Red stars) no matter how many times you refresh the page. The ratings service is being by reviews-v3 service.

Next, deny access to revies-v3 service by creating a rule, instance and handler.

```bash
[root@osc01 (istio-lab)policies]# kubectl apply -f 10-update-bookinfo-reviews-denyv3handler.yaml
handler.config.istio.io/denyreviewsv3handler created
instance.config.istio.io/denyreviewsv3request created
rule.config.istio.io/denyreviewsv3 created
```

Note: If using Istio v1.1.2 or before, apply the denyv3handler CRDs for reviews to be updated and take affect.

```bash
[root@osc01 (istio-lab)policies]# kubectl apply -f 11-create-bookinfo-reviews-denyv3handler-crd.yaml
denier.config.istio.io/denyreviewsv3handler created
checknothing.config.istio.io/denyreviewsv3request created
rule.config.istio.io/denyreviewsv3 configured
```

The deny rule states that any request matches will be denied if they come from `reviews` version 3 to the `ratings` service. It uses a `denier` adapater to deny requests coming from reviews-v3 or whatever service is identified. The deny adapater will show a pre-configured status code and message specified within the adapater configuration.

Refresh the product page, while logged out, reviews-v3 (red stars) will be blocked and the message: "Ratings service is currently unavailable" will show up. 

When logged in as `jason` or as any other user, you will only see reviews-v2 (black stars)

The above details confirm that the ratings service will only be available if logged in a user `jason`. 

### Attribute based whitelists or blacklists

Enable whitelist configuration, which is equivalent to `denier` configuration. This rule will reject requets from reviews-v3.

Remove the denier configuration & CRD from the reviews-v3 deny rule:

```bash
[root@osc01 (istio-lab)policies]# kubectl delete -f 10-update-bookinfo-reviews-denyv3handler.yaml
handler.config.istio.io "denyreviewsv3handler" deleted
instance.config.istio.io "denyreviewsv3request" deleted
rule.config.istio.io "denyreviewsv3" deleted
[root@osc01 (istio-lab)policies]# kubectl delete -f 11-create-bookinfo-reviews-denyv3handler-crd.yaml
denier.config.istio.io "denyreviewsv3handler" deleted
checknothing.config.istio.io "denyreviewsv3request" deleted
```

After both the YAMLs are delete, refresh the `productpage` without being logged in as a user. The productpage should show version-v3 (red stars). 

Apply configuration for `list` adapter that white-lists versions v1 and v2: 

```bash
[root@osc01 (istio-lab)policies]# kubectl apply -f 12-update-bookinfo-whitelist-v1v2.yaml
handler.config.istio.io/whitelist created
instance.config.istio.io/appversion created
rule.config.istio.io/checkversion created
```

Note: If using Istio v1.1.2 or before, apply the whitelist v1 & v2 CRDs for reviews to be updated and take affect.

```bash
[root@osc01 (istio-lab)policies]# kubectl apply -f 13-create-bookinfo-whitelist-v1v2-crd.yaml
listchecker.config.istio.io/whitelist created
listentry.config.istio.io/appversion created
rule.config.istio.io/checkversion configured
```

If you navigate to `productpage`, while logged in as `jason`, you will see reviews-v2 (black stars). While logged out, you will see reviews overall is not available. 

### IP based whitelists or blacklists

Istio supports whitelists and blacklists based IP address. It can be configured to accept or reject form a specific IP address or a subnect.

Verify access to `productpage` is available because after the below rules are applied, product page will not be available because request will be defined for a subnet of IP addresses.

Apply the configuration for `list` adapter that whitelists subnet "10.57.0.0\16" at the ingress gateway:

```bash
[root@osc01 (istio-lab)policies]# kubectl apply -f 14-create-bookinfo-deny-ip-crd.yaml
handler.config.istio.io/whitelistip created
instance.config.istio.io/sourceip created
rule.config.istio.io/checkip created
```

Note: If using Istio v1.1.2 or before, apply the deny subnet to `productpage` CRDs for updates to take affect.

```bash
[root@osc01 (istio-lab)policies]# kubectl apply -f 15-create-bookinfo-deny-subset-crd.yaml
listchecker.config.istio.io/whitelistip created
listentry.config.istio.io/sourceip created
rule.config.istio.io/checkip configured
```

!DIDN'T WORK
If the product page is refreshed, a whitelist error will come up to notify that the IP address is whitelisted.







## Traffic Management

Traffic management is done through mixer. How to route requests dynamically to multiple versions of a microservice?

Access /productpage and refresh it few times. The book review output contains star ratings and sometime it shows and and other times it does not. This is due to the fact that Istio routes requests to all available versions in a round robin fashion.

Route all traffic (100%) to only one version of rating microservice.

1. Make a request routing rule that all traffic is sent only to v1 of the microservice. Show and explain.

2. Set a rule to route traffic to v2 of the rating service based upon a particular user identity

3. Explain Istio L7 routing features

4. Explain what traffic shifting - how to gradually migrate traffic from one version of a microservice to another. As an example : migrate from older version to a new version. Create sequence of rules that route a percentage of traffic to one service.



