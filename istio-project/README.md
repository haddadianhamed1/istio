1) Install ISTIO
```
kubectl apply -f install/kubernetes/istio-demo.yaml

kubectl get svc -n istio-system
kubectl get pods -n istio-system
```
2) Label namespace
```
kubectl label namespace default istio-injection=enabled
```

3) Install application
```
kubectl apply -f bookinfo.yaml

kubectl get pods
NAME                             READY     STATUS    RESTARTS   AGE
details-v1-6764bbc7f7-dgclv      2/2       Running   0          35s
productpage-v1-54b8b9f55-rtnm7   2/2       Running   0          34s
ratings-v1-7bc85949-r8crp        2/2       Running   0          35s
reviews-v1-fdbf674bb-h7dfw       2/2       Running   0          35s
reviews-v2-5bdc5877d6-ln5br      2/2       Running   0          34s
reviews-v3-dd846cc78-5mpgp       2/2       Running   0          34s

kubectl get services
NAME          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
details       ClusterIP   100.71.72.58     <none>        9080/TCP   56s
kubernetes    ClusterIP   100.64.0.1       <none>        443/TCP    24m
productpage   ClusterIP   100.66.35.44     <none>        9080/TCP   54s
ratings       ClusterIP   100.68.203.147   <none>        9080/TCP   55s
reviews       ClusterIP   100.67.0.176     <none>        9080/TCP   55s
```

4) Deploy gateway
```
kubectl apply -f bookinfo-gateway.yaml

kubectl get gateway
NAME               AGE
bookinfo-gateway   23s

```

5) Get the gateway IP
```
kubectl get svc istio-ingressgateway -n istio-system

```
