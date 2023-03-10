# Kubernetes_deploy_cluster_manually_on_EC2_instances_AWS

[ Kubernetes ] Deploy a Kubernetes cluster manually on EC2 instances using AWS console and terminal

## 1. Master Instance

### 1.1. Create EC2 instances (I created two, can be more):

- Go to the AWS console, click on EC2 and then launch instance.
- Select an Amazon Machine Image (AMI) for the instances. It is recommended to use an Amazon Linux 2 AMI for Kubernetes. 
- Choose an instance type and configure other settings as required. Minimum suitable to Kubernetes is "t2.medium" (which has 2 vCPUs).
- I created 1 master and 1 worker. But the quantity can be more.
- Create or select a security group that allows traffic to and from the instance on ports required by Kubernetes. In more details on ports here: https://kubernetes.io/docs/reference/networking/ports-and-protocols/
- SSH to the master instance.


<img width="700" alt="Screenshot 2023-03-04 at 20 54 10" src="https://user-images.githubusercontent.com/104728608/222928300-89729d2f-709c-41ea-8114-eded3a5bcc1a.png">

<img width="700" alt="Screenshot 2023-03-04 at 20 21 00" src="https://user-images.githubusercontent.com/104728608/222927284-d6c025d6-dd0c-490c-b4b0-21cdc6fed8b1.png">

<img width="700" alt="Screenshot 2023-03-04 at 20 30 18" src="https://user-images.githubusercontent.com/104728608/222927394-e0b02992-49ec-4400-8c67-78d8d5e78da2.png">

### 1.2. Install Docker on the master instance, start and enable Docker to launch on boot:

```
sudo yum install docker -y
sudo systemctl start docker
sudo systemctl enable docker
```

### 1.3. Install Kubernetes on the instance:

- Add the Kubernetes repository:
```
sudo tee /etc/yum.repos.d/kubernetes.repo <<EOF
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```
- Install Kubernetes:
```
sudo yum install -y kubelet kubeadm kubectl
```
```
sudo systemctl start kubelet.service
sudo systemctl enable kubelet.service
```

### 1.4. Initialize the Kubernetes cluster:
- Run the following command to initialize the cluster:
```
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```
- Note down the kubeadm join command that is displayed on the console as you will need it later to join worker nodes to the cluster.
<img width="700" alt="Screenshot 2023-03-04 at 20 39 16" src="https://user-images.githubusercontent.com/104728608/222927853-cf359cec-2e3c-44c1-a4c8-4f4eefbb1b2f.png">

### 1.5. Configure kubectl:
Run the following commands to configure kubectl:
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
### 1.6. Install a pod network add-on:

Flannel is a popular networking solution for Kubernetes that allows pods running on different nodes to communicate with each other.

Run the following command to install the Flannel pod network add-on:

```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```
<br><br><br><br>

## 2. Worker Instances (install on each)

### 2.0. SSH to the worker instance.

<img width="700" alt="Screenshot 2023-03-04 at 20 52 43" src="https://user-images.githubusercontent.com/104728608/222928282-b31c7907-f5df-41eb-93b4-3b82be4be00b.png">

<img width="700" alt="Screenshot 2023-03-04 at 20 57 35" src="https://user-images.githubusercontent.com/104728608/222928397-adecc68f-fe8d-49cf-b264-59a17d53f794.png">

### 2.1. Install Docker on the instance, start and enable Docker to launch on boot:

```
sudo yum install docker -y
sudo systemctl start docker
sudo systemctl enable docker
```

### 2.2. Install Kubernetes on the instance:

- Add the Kubernetes repository:
```
sudo tee /etc/yum.repos.d/kubernetes.repo <<EOF
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```
- Install Kubernetes:
```
sudo yum install -y kubelet kubeadm kubectl
```
```
sudo systemctl start kubelet.service
sudo systemctl enable kubelet.service
```

<br><br>

## 3. Test
### 3.1. Join the EC2 instance to the Kubernetes cluster:

MASTER NODE - run the following command to get the kubeadm join command:

```
sudo kubeadm token create --print-join-command
```


<img width="1024" alt="Screenshot 2023-03-04 at 21 08 30" src="https://user-images.githubusercontent.com/104728608/222929784-ca4260ca-96a5-40d6-bece-e3256f7da78b.png">

- Copy the kubeadm join command that is displayed on the console.


<br><br>
WORKER NODE - run the kubeadm join command to join the node to the cluster:

```
kubeadm join 172.31.84.144:6443 --token z4iu5e.fp32zfbzh8sdtsjp --discovery-token-ca-cert-hash sha256:651e9007c0d788d7eef1c02ca0882456134e1ec4d1e3cc0e93973ec2a4399bf7
```

### 3.2. Result:
- this is a success message on the WORKER NODE after launching the "join" command

<img width="1024" alt="Screenshot 2023-03-04 at 21 14 41" src="https://user-images.githubusercontent.com/104728608/222958934-5b5c22bc-74af-438d-a2da-bd9ac4388073.png">


<br><br>

```
kubectl get nodes
kubectl get all
```
- this is a success message on the MASTER NODE after launching standard kubectl commands
<img width="1024" alt="Screenshot 2023-03-04 at 21 22 23" src="https://user-images.githubusercontent.com/104728608/222929829-803aeb00-ab14-4cd3-8154-e5ee81f9ae16.png">

