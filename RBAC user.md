## Task 2: Allowing the User to access Namespace using  Role Binding
#### Prepare user
switch to the root user, create a new user named **test-user**, and switch to the newly created user's home directory.
```
sudo su
```
Create user and set password with below command:
```
adduser test-user
```
```
su test-user
```
```
cd /home/test-user
```
#### Generate TLS Certificates: 
Create a directory named cert to store TLS certificates. Then we generate a private key (test-user.key) and a Certificate Signing Request (CSR) (test-user.csr) for the user test-user.

```
mkdir cert && cd cert
```
```
openssl genrsa -out test-user.key 2048
```
```
openssl req -new -key test-user.key -out test-user.csr -subj "/CN=test-user/O=finance"
```
Verify the above created certificate files:
```
ls -l
```

Switch back to the  **root** user and jump to cert directory
Note:-Either run "exit" or "su root" (if root passswor is set) command to switch root user
```
exit
```
```
cd /home/test-user/cert/
```
Sign the CSR to generate a certificate (test-user.crt) using the Kubernetes cluster's CA certificate and key.
```
openssl x509 -req -in test-user.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out test-user.crt -days 500
```
Change the ownership of the certificate file to the test-user
```
chown test-user:test-user test-user.crt
```
Now switch to test-user again
```
su test-user
```
Set the credentials for the  test-user by specifying the user's certificate and key
```
kubectl config set-credentials test-user --client-certificate=/home/test-user/cert/test-user.crt --client-key=/home/test-user/cert/test-user.key
```
Set the context for the user to ensure they are associated with the correct cluster. 
```
kubectl config set-context test-user-context --cluster kubernetes --namespace devops --user test-user
```
```
kubectl config use-context test-user-context
```
Copy Paste the "Cluster info"  from root users config file to the ***/home/test-user/.kube/config*** file as mention below  

> - cluster:                 
>       certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCVENDQWUyZ0F3SUJBZ0lJVmh6dlZVYll2eDh3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TkRBek1UZ3dNalUzTWpaYUZ3MHpOREF6TVRZd016QXlNalphTUJVeApFekFSQmdOVkJBTVRDbXQxWW1WeWJtVjBaWE13Z2dFaU1BMEdDU3FHU0liM0RRRUJBUVVBQTRJQkR3QXdnZ0VLCkFvSUJBUUR1ZkYrRmVJWDdkR0MrZEhSeXY0czk0eld3JYeDF0VWpSWE1uZ1o2N24wVkEvQnNYNHExb29qZFZnUVgvQVFHdGFXb2ZUTkdkRgpDd2JTT2FyTm5vejMwZDBYVGNsT2IybVpDMEdUNW5POE1WczNDbW5pVGl6eTJ5MkxpdjhsYk03bFJwWjQ5ajR3CnRTU2NVSWZKS3Z6ZzBIUEwyMlhpYWtuK2FXbk9idHo0SlFHZWlEQ3dRR2JXMEYrbHBsbVEycmM0b0VHYWlxWHkKeUVNejNiejNFcyt5Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K  
>         server: https://172.31.2.53:6443 
>    name: kubernetes
>   
```
vi /home/test-user/.kube/config
```

Finally, view the kubeconfig file to ensure the changes are reflected and display current contexts
```
cat /home/test-user/.kube/config
```
```
kubectl config get-contexts
```

#### Prepare Namespace and grant access to test-user by configuring RBAC
Switch back to the root user  
Note:-Either run "exit" or "su root" (if root passswor is set) command to switch root user
```
exit
```
create a namespace named "devops", and deploy a pod within that namespace
```
kubectl create ns devops
```
```
kubectl -n devops run ng-pod --image=nginx --port=80
```
Create a role  named "test-user-role" that grants permissions to list, create, and delete pods and services within the devops namespace
```
kubectl -n devops create role dev-role --verb=list,create,delete --resource=pods,services
```
Now create a role binding that associates the role with the test-user
```
kubectl -n devops create rolebinding dev-rb --role=dev-role --user=test-user
```
Verify the Role and RoleBinding created in the previous step
```
kubectl -n devops get role,rolebinding
```
```
kubectl -n devops describe role
```
```
kubectl -n devops describe rolebinding
```

Switch back to the test-user, and verify that the user has access to the devops namespace by listing the pods within that namespace. Additionally, we run a command to create a new pod (pod-1) within the devops namespace to further confirm the user's permissions.

```
su test-user
```
```
kubectl get pod -n devops
```
```
kubectl -n devops run dev-pod --image=nginx --port=80 --expose
```
