####  Deploy nginx-ingress-controller

deploys the nginx-ingress-controller using the YAML manifest provided in the URL
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.0/deploy/static/provider/cloud/deploy.yaml
```
Verify Deployment and pods in the namespace:
```
kubectl get ns
```
```
kubectl -n ingress-nginx get all
```
View Controller Logs, If needed, view the logs of the nginx-ingress-controller pod:
```
kubectl logs <nginx-ingress-controller-pod-name> -n ingress-nginx
```
#### Create a Namespace and Deploy Applications:
creates a new namespace named ***ingress-ns*** for the application deployment:  
```
kubectl create ns ingress-ns
```
Deploy HTTPD and Nginx deployments:
```
kubectl -n ingress-ns create deployment httpd-dep --image httpd --port 80 --replicas 2
```
```
kubectl -n ingress-ns create deployment nginx-dep --image nginx --port 80 --replicas 2
```
Verify the deployied application:
```
kubectl -n ingress-ns get deploy
```
```
kubectl -n ingress-ns get pods
```

Expose the deployments as services:
```
kubectl -n ingress-ns expose deployment httpd-dep --port 80 --name httpd-svc

```
```
kubectl -n ingress-ns expose deployment nginx-dep --port 8080 --name nginx-svc
```

Verify the created services and their end points:
```
kubectl -n ingress-ns get svc
```
```
kubectl -n ingress-ns get ep
```
#### Set Up Ingress Rules
Create a file named ***ingressrule.yaml*** and copy the provided Ingress rule configuration into it:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
 annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
 namespace: ingress-ns
 name: rewrite
spec:
 ingressClassName: nginx
 rules:
 - http:
    paths:
    - path: /
      pathType: Prefix
      backend:
       service:
         name: httpd-svc
         port:
          number: 80
    - path: /test
      pathType: Prefix
      backend:
       service:
         name: nginx-svc
         port:
          number: 8080
```
Apply the Ingress rule configuration from the file:
```
kubectl apply -f ingressrule.yaml
```
Verify the created Ingress:
```
kubectl -n ingress-ns get ingress
```
```
kubectl -n ingress-ns describe ingress
```
#### Verify Ingress Connectivity
```

curl -kv <ingress-address>/
```
```
curl -kv <ingress-address>/test

```



