## Network Policy

### Task 1: Check the network connectivity between pods in different namespaces
Create 2 namespaces
```
kubectl create ns devops

```
```
kubectl create ns finance
```
Create pod in devops namespace
```
kubectl -n devops run ng-pod --image nginx --port 80 --expose
```
Verify the pod and service in the devops namespace
```
kubectl -n devops get all
```
```
kubectl -n devops get po -o wide
```
Create pod in finance namespace
```
kubectl -n finance run pod2 --image centos -- sleep 7000
```
Verify the pod in the finance namespace
```
kubectl -n devops get po
```

Enter the pod in finance namespace  and check its connectivity with a pod in devops namespace
```
kubectl -n finance exec -it pod2 -- bash
```
Curl with the IP address of ng-pod or service of ng-pod
```
 curl <IP of the ng-pod>
 
```
```
curl <IP of the ng-pod service>
```
```
exit
```
**Note** You can able to curl the ng-pod 

### Task 2: Ingress Network policy with Pod labels 

Create a network policy which uses labels to deny all ingress traffic 
```
vi np-deny-all.yml
```
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  namespace: devops
  name: backend-policy
spec:
  podSelector:
    matchLabels:
      run: ng-pod
  ingress: []
```
```
kubectl apply -f np-deny-all.yml
```
```
kubectl -n devops get networkpolicies
```
```
kubectl -n devops describe networkpolicy backend-policy
```
Now enter into the  pod2 in finance namespace again and verify that it cannot curl the ng-pod from devops namespace

```
kubectl -n finance exec -it pod2 -- bash
```
Curl with the IP address of ng-pod or service of ng-pod
```
 curl <IP of the ng-pod>
```
```
curl <IP of the ng-pod service>
```
It will show timeout
```
exit
```
Modify the network policy to allow traffic with matching namespace labels 
Inspect the network policy and notice the selectors
```
vi np-deny-all.yml
```
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  namespace: devops
  name: backend-policy
spec:
  podSelector:
    matchLabels:
      run: ng-pod
  ingress: 
  - from:
    - namespaceSelector:
        matchLabels:
          project: myproject  
```
Before applying this yaml describe networkpolicies to check rule
```
kubectl -n devops describe networkpolicy backend-policy
```
```
kubectl apply -f np-deny-all.yml
```
```
kubectl describe networkpolicies backend-policy
```
Now enter into the  pod2 in finance namespace again and verify that it cannot curl the ng-pod from devops namespace

```
kubectl -n finance exec -it pod2 -- bash
```
Curl with the IP address of ng-pod or service of ng-pod
```
 curl <IP of the ng-pod>
```
```
curl <IP of the ng-pod service>
```
You can able to access
```
exit
```



