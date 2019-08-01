# Commands used in Chapter - Security

1.0

```
curl http://httpbin.istio-lab.svc.cluster.local:8000/status/418
```

2.0

```
curl -s https://api.github.com/repos/smallstep/cli/releases/latest | grep tag_name
```

3.0

```
cd ~/

curl -LOs https://github.com/smallstep/cli/releases/download/v0.10.1/step_0.10.1_linux_amd64.tar.gz
```

4.0

```
tar xvfz step_0.10.1_linux_amd64.tar.gz

sudo mv step_0.10.1/bin/step /bin
```

5.0

```
mkdir -p ~/step

cd ~/step

step certificate create --profile root-ca "My Root CA" root-ca.crt root-ca.key
```

6.0

```
step certificate create istio.io istio.crt istio.key --profile intermediate-ca --ca ./root-ca.crt --ca-key ./root-ca.key

```

7.0

```
step certificate create httpbin.istio.io httpbin.crt httpbin.key --profile leaf --ca istio.crt --ca-key istio.key --no-password --insecure
```

8.0

```
step certificate create bookinfo.istio.io bookinfo.crt bookinfo.key --profile leaf --ca istio.crt --ca-key istio.key --no-password --insecure
```

9.0

```
step certificate inspect bookinfo.crt --short
```

10.0

```
step certificate verify bookinfo.crt -roots istio.crt

echo $? 
```

11.0

```
export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].port}') ; echo $INGRESS_PORT

export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress..ip}') ; echo $INGRESS_HOST
```

12.0

```
if ! grep -q bookinfo.istio.io /etc/hosts ; then echo "$INGRESS_HOST bookinfo.istio.io bookinfo" | sudo tee -a /etc/hosts; fi

if ! grep -q httpbin.istio.io /etc/hosts ; then echo "$INGRESS_HOST httpbin.istio.io httpbin" | sudo tee -a /etc/hosts; fi

cat /etc/hosts
```

13.0

```
ping -c4 bookinfo.istio.io

ping -c4 httpbin.istio.io
```

14.0

```
kubectl -n istio-system get deploy istio-ingressgateway -o yaml > ~/servicemesh/istio-ingressgateway-non-sds.yaml

cd ~/istio-$ISTIO_VERSION

$ helm template install/kubernetes/helm/istio/ --name istio --namespace istio-system -x charts/gateways/templates/deployment.yaml --set gateways.istio-egressgateway.enabled=false --set gateways.istio-ingressgateway.sds.enabled=true > ~/servicemesh/istio-ingressgateway-sds.yaml

kubectl -n istio-system apply -f ~/servicemesh/istio-ingressgateway-sds.yaml 
```

15.0

```
kubectl -n istio-system create secret generic httpbin-keys --from-file=key=$HOME/step/httpbin.key --from-file=cert=$HOME/step/httpbin.crt 
```

16.0

```
kubectl -n istio-system create secret generic bookinfo-keys --from-file=key=$HOME/step/bookinfo.key --from-file=cert=$HOME/step/bookinfo.crt
```

17.0

```
cd ~/istio
git checkout $ISTIO_VERSION
cd scripts/02-security
```

18.0

```
kubectl -n istio-system apply -f 01-add-bookinfo-https-to-mygateway.yaml
```

19.0

```
kubectl -n istio-system apply -f 02-create-virtual-service-for-httpbin.yaml
```

20.0

```
export INGRESS_GW=$(kubectl -n istio-system get pods -l istio=ingressgateway -o jsonpath='{.items[0].metadata.name}') ; echo $INGRESS_GW

kubectl -n istio-system delete pod $INGRESS_GW
```

21.0

```
curl -HHost:httpbin.istio.io --resolve httpbin.istio.io:$INGRESS_PORT:$INGRESS_HOST --cacert $HOME/step/istio.crt https://httpbin.istio.io/status/418

curl -HHost:httpbin.istio.io --resolve httpbin.istio.io:$INGRESS_PORT:$INGRESS_HOST --cacert $HOME/step/istio.crt https://httpbin.istio.io/ip

```

22.0

```
kubectl -n istio-system apply -f 03-create-virtual-service-for-bookinfo.yaml 
```

23.0

```
cd ~/step

step certificate inspect httpbin.crt --short
```

24.0

```
kubectl -n istio-system delete secret httpbin-keys 
```

25.0

```
step certificate create httpbin.istio.io httpbin.crt httpbin.key --profile leaf --ca istio.crt --ca-key istio.key --no-password --insecure
```

26.0

```
kubectl -n istio-system create secret generic httpbin-keys --from-file=key=$HOME/step/httpbin.key --from-file=cert=$HOME/step/httpbin.crt 
```

27.0

```
curl -HHost:httpbin.istio.io --resolve httpbin.istio.io:$INGRESS_PORT:$INGRESS_HOST --cacert $HOME/step/istio.crt https://httpbin.istio.io/ip
```

28.0
```
step certificate create httpbin.istio.io client.crt client.key --profile leaf --ca istio.crt --ca-key istio.key --no-password --insecure --kty RSA --size 2048
```

29.0

```
step certificate bundle root-ca.crt istio.crt ca-chain.crt
```

30.0

```
kubectl -n istio-system delete secret httpbin-keys


kubectl -n istio-system create secret generic httpbin-keys --from-file=key=$HOME/step/httpbin.key --from-file=cert=$HOME/step/httpbin.crt --from-file=cacert=$HOME/step/ca-chain.crt
```

31.0

```
kubectl -n istio-system apply -f 04-add-mutual-TLS-to-bookinfo-https-to-mygateway.yaml
```

32.0

```
export INGRESS_GW=$(kubectl -n istio-system get pods -l istio=ingressgateway -o jsonpath='{.items[0].metadata.name}') ; echo $INGRESS_GW

kubectl -n istio-system delete pod $INGRESS_GW
```

33.0

```
curl -v -HHost:httpbin.istio.io --resolve httpbin.istio.io:$INGRESS_PORT:$INGRESS_HOST --cacert $HOME/step/istio.crt https://httpbin.istio.io/status/418
```

34.0

```
curl -HHost:httpbin.istio.io --resolve httpbin.istio.io:$INGRESS_PORT:$INGRESS_HOST --cacert $HOME/step/ca-chain.crt --cert $HOME/step/client.crt --key $HOME/step/client.key https://httpbin.istio.io/status/418
```

35.0
```
cd ~/istio-$ISTIO_VERSION

helm template install/kubernetes/helm/istio/ --name istio --namespace istio-system -x charts/nodeagent/templates/serviceaccount.yaml --set nodeagent.enabled=true > ~/servicemesh/istio-node-agent-sa.yaml

kubectl -n istio-system apply -f ~/servicemesh/istio-node-agent.yaml

helm template install/kubernetes/helm/istio/ --name istio --namespace istio-system -x charts/nodeagent/templates/daemonset.yaml --set nodeagent.enabled=true --set nodeagent.env.CA_PROVIDER=Citadel --set nodeagent.env.CA_ADDR=istio-citadel:8060 > ~/servicemesh/istio-node-agent.yaml

kubectl -n istio-system apply -f ~/servicemesh/istio-node-agent-sa.yaml
```

36.0

```
kubectl -n istio-system get pods -l app=nodeagent
```

37.0

```
PRODUCT_PAGE=$(kubectl -n istio-lab get pods -l app=productpage -o jsonpath={.items..metadata.name}) ; echo $PRODUCT_PAGE

istioctl authn tls-check $PRODUCT_PAGE.istio-lab productpage.istio-lab.svc.cluster.local
```

38.0

```
kubectl get meshpolicies default -o yaml
```

39.0
```
kubectl -n istio-lab apply -f 05-create-mtls-bookinfo-destination-rules.yaml 
```

40.0
```
kubectl -n istio-lab apply -f 06-create-mtls-httpbin-destination-rules.yaml 
```

41.0
```
kubectl -n istio-lab apply -f 07-create-mtls-for-istio-lab-namespace.yaml 
```

42.0
```
export RATING_POD=$(kubectl -n istio-lab get pods -l app=ratings -o jsonpath='{.items[0].metadata.name}') ; echo $RATING_POD

istioctl authn tls-check $RATING_POD.istio-lab ratings.istio-lab.svc.cluster.local
```

43.0
```
kubectl -n istio-system apply -f 08-disable-mtls-for-kube-apiserver.yaml
```

44.0
```
curl -HHost:httpbin.istio.io --resolve httpbin.istio.io:$INGRESS_PORT:$INGRESS_HOST --cacert $HOME/step/ca-chain.crt --cert $HOME/step/client.crt --key $HOME/step/client.key https://httpbin.istio.io/status/418

```

45.0
```
cp -r ~/step /tmp && chmod -R 777 /tmp/step
```

46.0
```
certutil -d sql:$HOME/.pki/nssdb -A -n httpbin.istio.io -i /tmp/step/root-ca.crt -t "TC,,"
```

47.0
```
openssl pkcs12 -export -clcerts  -inkey /tmp/step/client.key -in /tmp/step/client.crt -out httpbin.istio.io.p12 -passout pass:password -name "Key pair for httpbin.istio.io"
```

48.0
```
pk12util -i httpbin.istio.io.p12 -d sql:$HOME/.pki/nssdb -W password
```

49.0
```
certutil -d sql:$HOME/.pki/nssdb -L
```

50.0
```
cd ~/istio/scripts/02-security
```

51.0
```
kubectl -n istio-lab apply -f 09-create-clusterrbac-config.yaml
```

52.0
```
kubectl -n istio-lab apply -f 10-create-service-role.yaml
```

53.0
```
kubectl -n istio-lab apply -f 11-create-service-role-binding.yaml
```

54.0
```
kubectl -n istio-lab delete -f 11-create-service-role-binding.yaml

kubectl -n istio-lab delete -f 10-create-service-role.yaml
```

55.0
```
kubectl -n istio-lab apply -f 12-create-service-role-productpage.yaml

```

56.0
```
kubectl -n istio-lab apply -f 13-create-service-role-binding-productpage.yaml
```

57.0
```
kubectl -n istio-lab apply -f 14-create-service-role-details-reviews.yaml
```

58.0
```
kubectl -n istio-lab apply -f 15-create-service-role-binding-details-reviews.yaml
```

59.0
```
kubectl -n istio-lab create sa bookinfo-productpage
```

60.0
```
kubectl -n istio-lab set sa deployment productpage-v1 bookinfo-productpage
```

61.0
```
kubectl -n istio-lab get deployment productpage-v1 -o yaml | grep bookinfo-productpage
```

62.0
```
kubectl -n istio-lab apply -f 16-apply-service-role-binding-details-reviews.yaml 
```

63.0
```
kubectl -n istio-lab create sa bookinfo-reviews
```

64.0
```
kubectl -n istio-lab set sa deployment reviews-v1 bookinfo-reviews

kubectl -n istio-lab set sa deployment reviews-v2 bookinfo-reviews

kubectl -n istio-lab set sa deployment reviews-v3 bookinfo-reviews
```

65.0
```
kubectl -n istio-lab apply -f 17-create-service-role-ratings.yaml 
```

66.0
```
kubectl -n istio-lab apply -f 18-create-service-role-binding-ratings.yaml
```

67.0
```
kubectl -n istio-lab apply -f 19-create-sa-ratings-v2.yaml 
```

68.0
```
kubectl -n istio-lab get dr ratings -o yaml | grep -A6 subsets:
```

69.0
```
kubectl -n istio-lab get vs ratings -o yaml | grep -B1 subset: 
```

70.0
```
kubectl -n istio-lab patch vs ratings --type json -p '[{"op":"replace","path":"/spec/http/0/route/0/destination/subset","value": "v2"}]'
```

71.0
```
kubectl -n istio-lab get vs ratings -o yaml | grep -B1 subset:
```

72.0
```
kubectl -n istio-lab apply -f 20-deploy-mongodb-service.yaml 
```

73.0
```
kubectl -n istio-lab get pods -l app=mongodb
```

74.0
```
kubectl -n istio-lab apply -f 21-create-service-role-mongodb.yaml 
```

75.0
```
kubectl -n istio-lab apply -f 22-create-service-role-binding-mongodb.yaml 
```

76.0
```
export RATINGS_POD=$(kubectl -n istio-lab get pods -l app=ratings -o jsonpath='{.items[0].metadata.name}') ; echo $RATINGS_POD
```

77.0
```
istioctl authn tls-check $RATINGS_POD.istio-lab mongodb.istio-lab.svc.cluster.local
```

78.0

```
kubectl -n istio-lab apply -f 23-create-mongodb-destination-rule.yaml
```

79.0
```
istioctl authn tls-check $RATINGS_POD.istio-lab mongodb.istio-lab.svc.cluster.local
```

80.0
```
export MONGO_POD=$(kubectl -n istio-lab get pod -l app=mongodb -o jsonpath='{.items..metadata.name}') ; echo $MONGO_POD

cat << EOF | kubectl -n istio-lab exec -i -c mongodb $MONGO_POD -- mongo
use test
db.ratings.find().pretty()
db.ratings.update({"rating": 5},{\$set:{"rating":1}})
db.ratings.update({"rating": 4},{\$set:{"rating":3}})
db.ratings.find().pretty()
exit
EOF
```

81.0
```
kubectl -n istio-lab apply -f 24-create-service-role-binding-httpbin.yaml
```

