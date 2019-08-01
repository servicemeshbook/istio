# Commands used in Chapter - Traffic Management

1.0

```
cd # Switch to home directory 
git clone https://github.com/servicemeshbook/istio
cd istio
git checkout $ISTIO_VERSION # Switch to branch version that we are using
```

2.0

```
cd scripts/01-traffic-management
```

3.0

```
kubectl -n istio-system apply -f 00-create-gateway.yaml
```

4.0

```
kubectl -n istio-system get svc istio-ingressgateway
```

5.0

```
export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress..ip}') ; echo $INGRESS_HOST
```

6.0

```
curl -v http://192.168.142.249
```

7.0

```
kubectl -n istio-system apply -f 01-create-virtual-service.yaml
```

8.0

```
kubectl -n istio-lab get ep | grep reviews
```

9.0

```
kubectl -n istio-lab get pods -o=custom-columns=NAME:.metadata.name,POD_IP:.status.podIP
```

10.0

```
kubectl -n istio-system get gateway
kubectl -n istio-system get vs
```

11.0

```
kubectl -n istio-lab get pods -o=custom-columns=NAME:.metadata.name,POD_IP:.status.podIP
```

12.0

```
export PRODUCTPAGE_POD=$(kubectl -n istio-lab get pods -l app=productpage -o jsonpath='{.items..status.podIP}') ; echo $PRODUCTPAGE_POD

curl -s http://$PRODUCTPAGE_POD:9080 | grep title
```

13.0

```
kubectl -n istio-lab get svc -o custom-columns=NAME:.metadata.name,CLUSTER_IP:.spec.clusterIP
```

14.0

```
curl -s http://10.105.226.61:9080 | grep title
```

15.0

```
kubectl -n istio-lab get ep productpage
```

16.0

```
dig +short productpage.istio-lab.svc.cluster.local @10.96.0.10
```

17.0

```
kubectl -n istio-lab edit svc productpage
```

18.0

```
kubectl -n istio-lab get svc
```

19.0

```
kubectl get nodes
```

20.0

```
kubectl -n istio-lab get pods -l app=reviews --show-labels
```

21.0

```
kubectl -n istio-lab apply -f 02-create-destination-rules.yaml
```

22.0

```
kubectl -n istio-lab apply -f 03-create-virtual-service-for-v1.yaml
```

23.0

```
kubectl -n istio-lab apply -f 04-identity-based-traffic-routing.yaml
```

24.0

```
kubectl -n istio-lab apply -f 05-chrome-browser-traffic-routing.yaml
```

25.0

```
kubectl -n istio-lab apply -f 06-canary-deployment-weight-based-routing.yaml
```

26.0
```
echo $INGRESS_HOST

curl -s http://$INGRESS_HOST/productpage?[1-1000] | grep -c "full stars"
```

27.0
```
kubectl -n istio-lab apply -f 07-move-canary-to-production.yaml
```

28.0

```
curl -s http://$INGRESS_HOST/productpage?[1-1000] | grep -c "full stars"
```

29.0

```
kubectl -n istio-lab apply -f 08-inject-http-delay-fault.yaml
```

30.0

```
kubectl -n istio-lab apply -f 09-inject-http-abort-fault.yaml
```

31.0

```
kubectl -n istio-lab apply -f 10-set-request-timeout.yaml
```

32.0

```
kubectl -n istio-lab apply -f 11-inject-latency.yaml
```

33.0

```
kubectl -n istio-lab delete -f 03-create-virtual-service-for-v1.yaml

kubectl -n istio-lab create -f 03-create-virtual-service-for-v1.yaml
```

34.0

```
kubectl -n istio-lab apply -f 12-modify-productpage-destination-rule-for-circuit-breaker.yaml
```

35.0

```
kubectl -n istio-lab apply -f 13-install-fortio-testing-tool.yaml
```

36.0

```
kubectl -n istio-lab get deploy fortio-deploy
```

37.0

```
kubectl -n istio-lab get pods -l app=fortio
```

38.0

```
export FORTIO_POD=$(kubectl -n istio-lab get pods -l app=fortio --no-headers -o custom-columns=NAME:.metadata.name) ; echo $FORTIO_POD
```

39.0

```
kubectl -n istio-lab exec -it $FORTIO_POD -c fortio /usr/bin/fortio -- load -c 1 -qps 0 -n 1 -loglevel Warning http://productpage:9080
```

40.0

```
kubectl -n istio-lab exec -it $FORTIO_POD -c fortio /usr/bin/fortio -- load -c 3 -qps 0 -n 20 -loglevel Warning http://productpage:9080
```

41.0

```
kubectl -n istio-lab apply -f 02-create-destination-rules.yaml
```

42.0

```
kubectl -n istio-system get svc istio-ingressgateway -o custom-columns=Name:.metadata.name,EXTERNAL_IP:.status.loadBalancer.ingress[0].ip
```

43.0

```
export INGRESS_IP=$(kubectl -n istio-system get svc istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}') ; echo $INGRESS_IP

echo "$INGRESS_IP bookinfo.istio.io" | sudo tee -a /etc/hosts
```

44.0

```
kubectl -n istio-system apply -f 14-create-bookinfo-virtual-service.yaml
```

45.0

```
curl -s http://bookinfo.istio.io | grep title
```

46.0

```
kubectl -n istio-system get cm istio -o yaml | grep -m1 -o "mode: ALLOW_ANY"
```

47.0

```
export RATING_POD=$(kubectl -n istio-lab get pods -l app=ratings -o jsonpath='{.items..metadata.name}') ; echo $RATING_POD
```

48.0

```
kubectl -n istio-lab exec -it -c ratings $RATING_POD -- curl -LI https://www.ibm.com | grep "HTTP/"

kubectl -n istio-lab exec -it -c ratings $RATING_POD -- curl -LI https://www.cnn.com | grep "HTTP/"
```

49.0

```
kubectl -n istio-system get cm istio -o yaml | sed 's/mode: ALLOW_ANY/mode: REGISTRY_ONLY/g' | kubectl replace -n istio-system -f -
```

50.0

```
kubectl -n istio-system get cm istio -o yaml | grep -m 1 -o "mode: REGISTRY_ONLY"
```

51.0

```
kubectl -n istio-lab exec -it -c ratings $RATING_POD -- curl -LI https://www.ibm.com | grep "HTTP/"
```

52.0

```
kubectl -n istio-lab exec -it -c ratings $RATING_POD -- curl -LI https://www.cnn.com | grep "HTTP/"
```

53.0

```
kubectl -n istio-lab apply -f 15-http-service-entry-for-httpbin.yaml
```

54.0

```
kubectl -n istio-lab apply -f 16-https-service-entry-for-ibm.yaml
```

55.0

```
kubectl -n istio-lab exec -it -c ratings $RATING_POD -- curl -LI https://www.ibm.com | grep "HTTP/"
```

56.0

```
RATING_POD=$(kubectl -n istio-lab get pods -l app=ratings -o jsonpath='{.items..metadata.name}') ; echo $RATING_POD

kubectl -n istio-lab exec -it -c ratings $RATING_POD -- curl http://httpbin.org/headers
```

57.0

```
kubectl -n istio-lab logs -c istio-proxy $RATING_POD | tail | grep curl
```

58.0

```
kubectl -n istio-lab exec -it -c ratings $RATING_POD -- curl -LI https://www.ibm.com | grep "HTTP/"
```

59.0

```
kubectl -n istio-lab exec -it -c ratings $RATING_POD -- curl -LI https://www.cnn.com | grep "HTTP/"
```

60.0

```
kubectl -n istio-lab apply -f 17-add-timeout-for-httpbin-virtual-service.yaml
```

61.0

```
time kubectl -n istio-lab exec -it -c ratings $RATING_POD -- curl -o /dev/null -s -w "%{http_code}\n" http://httpbin.org/delay/5
```

62.0

```
kubectl -n istio-lab apply -f 18-deploy-httpbin-v1.yaml
```

63.0

```
kubectl -n istio-lab apply -f 19-deploy-httpbin-v2.yaml
```

64.0

```
kubectl -n istio-lab apply -f 20-create-kubernetes-httpbin-service.yaml
```

65.0

```
kubectl -n istio-lab apply -f 21-create-destination-rules-subsets.yaml
```

66.0

```
kubectl -n istio-lab apply -f 22-create-httpbin-virtual-service.yaml
```

67.0

```
V1_POD=$(kubectl -n istio-lab get pod -l app=httpbin,version=v1 -o jsonpath={.items..metadata.name})

echo $V1_POD

kubectl -n istio-lab -c httpbin logs -f $V1_POD
```

68.0

```
V2_POD=$(kubectl -n istio-lab get pod -l app=httpbin,version=v2 -o jsonpath={.items..metadata.name})

echo $V2_POD

kubectl -n istio-lab -c httpbin logs -f $V2_POD

```

69.0

```
RATING_POD=$(kubectl -n istio-lab get pods -l app=ratings -o jsonpath='{.items..metadata.name}')

echo $RATING_POD

kubectl -n istio-lab exec -it $RATING_POD -c ratings -- curl http://httpbin:8000/headers | python -m json.tool
```

70.0

```
kubectl -n istio-lab apply -f 23-mirror-traffic-between-v1-and-v2.yaml
```

71.0

```
kubectl -n istio-lab exec -it $RATING_POD -c ratings -- curl http://httpbin:8000/headers | python -m json.tool
```

