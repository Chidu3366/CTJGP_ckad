## Bootstrap a Kubernetes Cluster using Kubeadm

To begin, log in to AWS Console.

### Task 1: Launching Instances on AWS

* `3` instances of `t2.medium` instance with OS version as `Ubuntu 22.04 LTS` in your preferred region.
* `Storage : 10 GB`
* Instead of opening all ports you can open these ports internally.
* `Type : Custom TCP`
* `Source type : Anywhere`
    |      Nodes	      |    Port Number	 |         Use Case                       |
    |---------------------|------------------|----------------------------------------|
    | Master, Workers	  |    `2379-2380`   |  Etcd Client API / Server API          |
    | Master              |       `6443`  	 |  Kubernetes API Server (Secure Port)   |
    | Master, Workers     |   `6782-6784`    |  Weave Net Server/Client API #CNI      |
    | Master, Workers     |   `10250-10255`	 |  Kubelet Communication                 |
    | Workers             |   `30000-32767`	 |  Reserved of NodePort Ips              |	   


* Add the below code in Advanced Details -> User data - optional.
```
#!/bin/bash
sudo apt-get update
sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common -y
sudo apt install docker.io -y
sudo usermod -aG docker ubuntu

sudo cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "1024m"
  },
  "storage-driver": "overlay2"
}
EOF

sudo systemctl daemon-reload
sudo systemctl restart docker

sudo echo "Environment=cgroup-driver=systemd/cgroup-driver=cgroupfs" >> /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
sudo echo 'deb http://apt.kubernetes.io/ kubernetes-xenial main' >> /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet=1.23.0-00 kubeadm=1.23.0-00 kubectl=1.23.0-00
```

* Launch the Instance.

### Task-2: Setting up Machines
* Task 2 can be skipped if the user data has been updated as shown in Task 1
* All steps in this task are to be performed on all the machines
* Connect all VMs with putty.
Switch to root.
```
sudo su
``` 

Download the script to install and configure Kubeadm using the link on all the 3 instances.
```
wget https://ckad.s3.ap-northeast-1.amazonaws.com/kubeadm-setup.sh
```
OR
Create kubeadm.sh script file
```
vi kubeadm-setup.sh
```
```
#!/bin/bash
sudo apt-get update
sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common -y
sudo apt install docker.io -y
sudo usermod -aG docker ubuntu

sudo cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "1024m"
  },
  "storage-driver": "overlay2"
}
EOF

sudo systemctl daemon-reload
sudo systemctl restart docker

sudo echo "Environment=cgroup-driver=systemd/cgroup-driver=cgroupfs" >> /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
sudo echo 'deb http://apt.kubernetes.io/ kubernetes-xenial main' >> /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet=1.23.0-00 kubeadm=1.23.0-00 kubectl=1.23.0-00

```
Save the file using "ESCAPE + :wq!"

Run the script to set up and configure kubeadm on all 3 instances.

```
bash kubeadm-setup.sh
```

### Task 3: Initializing the Cluster

rename the VM's as Master, Node1 and Node2 from the AWS Console.

Set the hostname to all three nodes as master, Node1, and Node2 in their respective terminals for easy understanding, by running the below command:

Connect to Master.

Switch to root.
```
sudo su
```
```
hostnamectl set-hostname Master
```
```
bash
```

Connect to Node1.

Switch to root.
```
sudo su
```
```
hostnamectl set-hostname Node1
```
```
bash
```

Connect to Node2.

Switch to root.
```
sudo su
```
```
hostnamectl set-hostname Node2
```
```
bash
```

Start kubeadm only on master
```
kubeadm init
```

If it runs successfully, it will provide a join command which can be used to join the master. Make a note of the highlighted part.

If you want to list and generate tokens again to join worker nodes, then follow the below steps.
```
kubeadm token list
kubeadm token create  --print-join-command
```
 
### Task 4: Joining a Cluster

Run the kubeadm join command in worker nodes, that was previously noted from the master node in the previous task.
```
kubeadm join --token <your_token> --discovery-token-ca-cert- hash <your_discovery_token>
```

Run the following commands to configure kubectl on master.
```
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config 
chown $(id -u):$(id -g) $HOME/.kube/config
```

View node information on the master. The nodes will not be ready.
```
kubectl get nodes
```

### Task 5: Deploy Container Networking Interface

Apply weave CNI (Container Network Interface) as shown below:
```
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
```
(reference link1:- https://www.weave.works/docs/net/latest/kubernetes/kube-addon/)

View nodes to see that they are ready.
```
kubectl get nodes
```

View all Pods including system Pods and see that dns and weave are running.
```
kubectl get pod -n kube-system
```

### Task 6: Create Pods
To check the API version of any resource
```
kubectl api-resources
```

Create a Pod called http based on a Docker image on the master.
```
kubectl run httpd --image=httpd
```

View the status of Pod and make sure it’s in the running state.
```
kubectl get pods
```

Get a shell to the container in the Pod using the Pod name from the previous step.
```
kubectl exec -it <pod_name> -- /bin/bash
``` 

Install curl in the container
```
apt update
apt install curl -y
```

Run curl on the localhost (container) to verify the http installation.
```
curl localhost 
exit
```

If you get the below `error` when running kubectl commands, execute the command given below.

`The connection to the server localhost:8080 was refused - did you specify the right host or port?`
```
export KUBECONFIG=/etc/kubernetes/admin.conf
```

#### *** DO NOT EXECUTE THE BELOW COMMAND UNTIL DOUBLY SURE ***
To remove the joins in your cluster
```
kubeadm reset
```


 

