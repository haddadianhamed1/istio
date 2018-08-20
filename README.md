https://github.com/kubernetes/minikube/blob/master/docs/drivers.md#xhyve-driver
install Hyperkit driver
```
curl -Lo docker-machine-driver-hyperkit https://storage.googleapis.com/minikube/releases/latest/docker-machine-driver-hyperkit \
&& chmod +x docker-machine-driver-hyperkit \
&& sudo cp docker-machine-driver-hyperkit /usr/local/bin/ \
&& rm docker-machine-driver-hyperkit \
&& sudo chown root:wheel /usr/local/bin/docker-machine-driver-hyperkit \
&& sudo chmod u+s /usr/local/bin/docker-machine-driver-hyperkit
```


# start minikube
```
minikube start --memory=8192 --cpus=4 --kubernetes-version=v1.10.0 \
--extra-config=controller-manager.cluster-signing-cert-file="/var/lib/localkube/certs/ca.crt" \   --extra-config=controller-manager.cluster-signing-key-file="/var/lib/localkube/certs/ca.key" --vm-driver=hyperkit --disk-size=10g --logtostderr
```

# Installing Istio
```
kubectl apply -f install/kubernetes/istio-demo.yaml

kubectl apply -f install/kubernetes/istio.yaml
```

# Installing sidecar
A) Manual sidecar Injection
Inject the sidecar into the deployment using the in-cluster configuration.
```
kubectl apply -f <(istioctl kube-inject --debug -f samples/bookinfo/kube/bookinfo.yaml)

istioctl kube-inject -f sleep.yaml | kubectl apply -f -
service "sleep" created
deployment.extensions "sleep" created
DB-MBP-K3G8WL:delete-istio hhaddadian$ kubectl get pods
NAME                     READY     STATUS        RESTARTS   AGE
sleep-6859f9d5cb-l877h   2/2       Running       0          18s
```

B) Automatically
```
kubectl api-versions | grep admissionregistration
admissionregistration.k8s.io/v1beta1

kubectl apply -f sleep.yaml
kubectl get deployment -o wide
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE       CONTAINERS   IMAGES       SELECTOR
sleep     1         1         1            0           14s       sleep        tutum/curl   app=sleep

#Label the default namespace with istio-injection=enabled
kubectl label namespace default istio-injection=enabled
kubectl get namespace -L istio-injection
NAME           STATUS    AGE       ISTIO-INJECTION
default        Active    40m       enabled
istio-system   Active    34m       
kube-public    Active    40m       
kube-system    Active    40m       

#Injection occurs at pod creation time. Kill the running pod and verify a new pod is created with the injected sidecar
kubectl delete pod sleep-86f6b99f94-5277g
```


# create secret
```
#kubectl create secret generic cacerts -n istio-system --from-file=samples/certs/ca-cert.pem \
#    --from-file=samples/certs/ca-key.pem --from-file=samples/certs/root-cert.pem \
#    --from-file=samples/certs/cert-chain.pem

kubectl create secret generic cacerts -n istio-system --from-file=cert.pem --from-file=key.pem
```

#Run the following OpenSSL command to generate your private key and public certificate. Answer the questions and enter the Common Name when prompted.
openssl req -newkey rsa:2048 -nodes -keyout ca-key.pem -x509 -days 365 -out ca-cert.pem
openssl x509 -text -noout -in ca-cert.pem

## Distributed tracing
# Zipkin
kubectl apply -f install/kubernetes/addons/zipkin.yaml
