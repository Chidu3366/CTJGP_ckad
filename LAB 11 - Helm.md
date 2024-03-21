### Task 1: Installing Helm 3 in Ubuntu Linux
setting up Helm on Kubernetes4
Run the following command to update the packages:
```
apt update
```
Download the Helm binary for Linux 386 architecture:
```
wget https://get.helm.sh/helm-v3.11.3-linux-386.tar.gz
```
Unpack the downloaded tarball: 
```
tar -xvzf helm-v3.11.3-linux-386.tar.gz
```
Check the contents of the extracted directory to ensure that the Helm binary is present:
```
ls
```
Move the Helm executable to the /bin directory to make it accessible system-wide:
 
```
mv helm /bin/
```
Confirm that Helm is installed correctly by checking its version:
```
helm version
```
### Task 2:  Commonly used Helm Commands

To check the version of Helm installed
```
helm version
```

To add a Helm chart repository to your local environment. Below we are adding two Helm Chart Repositories.
```
helm repo add bitnami https://charts.bitnami.com/bitnami 
```
To list the Helm chart repositories configured in your local environment.
```
helm repo list
```
To search for Helm charts related to WordPress in the configured Helm repositories
```
helm search repo wordpress
```
To install the WordPress application using the Bitnami Helm chart with the release name my-wordpress
```
helm install my-wordpress bitnami/wordpress
```
To list all the releases currently installed on your Kubernetes cluster
```
helm list
```


#### Verify that WordPress has been set up 
Verify that the pods are running.
```
kubectl get pods
```
Now Notice that MariaDB (database) pods are part of statefulset.
```
kubectl get sts
```
```
kubectl describe sts
```
Also Notice that the front end of wordpress are part of deployment
```
kubectl get deploy
```
View the services using the below commands. Make note of the EXTERNAL IP of the LoadBalancer service.
```
kubectl get svc
```
```
kubectl get svc --namespace default -w wordpress-1637763307
``` 
Open the browser and paste the service endpoint noted on the previous step. Observe that the WordPress site is up and running.

#### Cleanup
List the current helm release and delete it
```
helm ls
```

To uninstall the WordPress application deployed using Helm with the release name my-wordpress
```
helm uninstall my-wordpress
```
```
helm ls
```
 To remove a Helm repository from your local Helm client configuration
```
helm repo remove bitnami
```
To remove Helm package manager
```
sudo apt-get remove helm
```
 
