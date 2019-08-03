# Commands used in Chapter - Policy

1.0

```
cd ~/istio
cd scripts/03-policies
```

2.0

```
kubectl -n istio-system get cm istio -o jsonpath="{@.data.mesh}" | grep disablePolicyChecks
```

3.0

```
kubectl -n istio-system apply -f 01-create-quota-instance.yaml
```

4.0

```
kubectl -n istio-system apply -f 02-create-quotaspec.yaml
```

5.0

```
kubectl -n istio-system apply -f 03-create-quotaspecbinding.yaml
```

6.0
```
kubectl -n istio-system apply -f 04-create-memquota-handler.yaml
```

7.0
```
kubectl -n istio-system apply -f 05-create-quota-rule.yaml
```

8.0
```
kubectl -n istio-system apply -f 06-modify-memquota-handler.yaml
```

9.0
```
kubectl -n istio-lab apply -f 07-modify-reviews-virtual-service.yaml
```

10.0
```
kubectl -n istio-system apply -f 08-create-denier-handler.yaml
```

11.0
```
kubectl -n istio-system apply -f 09-create-check-nothing-instance.yaml 
```

12.0
```
kubectl -n istio-system apply -f 10-create-denier-rule.yaml 
```

13.0
```
kubectl -n istio-system delete -f 10-create-denier-rule.yaml 

kubectl -n istio-system delete -f 09-create-check-nothing-instance.yaml

kubectl -n istio-system delete -f 08-create-denier-handler.yaml 
```

14.0
```
kubectl -n istio-system apply -f 11-create-listchecker-handler.yaml
```

15.0
```
kubectl -n istio-system apply -f 12-create-listentry-instance.yaml 
```

16.0
```
kubectl -n istio-system apply -f 13-create-whitelist-rule.yaml 
```

17.0
```
kubectl -n istio-system apply -f 14-create-listchecker-handler.yaml
```

18.0
```
kubectl -n istio-system apply -f 15-create-listentry-instance.yaml 
```

19.0
```
kubectl -n istio-system delete -f 16-create-whitelist-rule.yaml

kubectl -n istio-system delete -f 15-create-listentry-instance.yaml

kubectl -n istio-system delete -f 14-create-listchecker-handler.yaml

kubectl -n istio-system delete -f 13-create-whitelist-rule.yaml

kubectl -n istio-system delete -f 12-create-listentry-instance.yaml

kubectl -n istio-system delete -f 11-create-listchecker-handler.yaml 
```

