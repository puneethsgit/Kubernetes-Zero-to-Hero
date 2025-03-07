# Kubernetes-Zero-to-Hero

## Introduction
This repository is created with the intent to make Kubernetes easy for beginners. This is a work-in-progress repository.

---

## Kubernetes Installation Using KOPS on AWS EC2
You can set up Kubernetes using KOPS either on an EC2 instance or your personal laptop.

### Prerequisites
Ensure the following dependencies are installed:
- Python 3
- AWS CLI
- kubectl
- KOPS

### Install Dependencies
#### Add Kubernetes Repository & Install Required Packages
```sh
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y python3-pip apt-transport-https kubectl
pip3 install awscli --upgrade
export PATH="$PATH:/home/ubuntu/.local/bin/"
```

### Install KOPS
```sh
curl -LO https://github.com/kubernetes/kops/releases/latest/download/kops-linux-amd64
chmod +x kops-linux-amd64
sudo mv kops-linux-amd64 /usr/local/bin/kops
```

---

## AWS IAM Permissions
Ensure your IAM user has the following permissions (Admin users already have these by default):
- `AmazonEC2FullAccess`
- `AmazonS3FullAccess`
- `IAMFullAccess`
- `AmazonVPCFullAccess`

### Configure AWS CLI
Run the following command and provide your AWS credentials:
```sh
aws configure
```

---

## Kubernetes Cluster Installation
Follow these steps carefully and read each command before executing.

### Step 1: Create an S3 Bucket for Storing KOPS Objects
```sh
aws s3api create-bucket --bucket kops-your-storage-bucket --region us-east-1
```

### Step 2: Create the Kubernetes Cluster
```sh
kops create cluster \
--name=demok8scluster.k8s.local \
--state=s3://kops-your-storage-bucket \
--zones=us-east-1a \
--node-count=1 \
--node-size=t2.micro \
--master-size=t2.micro \
--master-volume-size=8 \
--node-volume-size=8
```

Your `kops create cluster` command includes deprecated flags. Here's the corrected version using the updated flags:  

### **Fixed Command:**  
```sh
kops create cluster --name=puneethk8cluster.k8s.local \
  --state=s3://kops-puneeth-storage \
  --zones=us-east-1a \
  --node-count=1 \
  --node-size=t2.micro \
  --control-plane-size=t2.micro \
  --control-plane-volume-size=8 \
  --node-volume-size=8
```

### **Key Fixes:**  
✅ Replaced `--master-size` with `--control-plane-size`  
✅ Replaced `--master-volume-size` with `--control-plane-volume-size`  
✅ The warning about Gossip being deprecated means you should configure DNS properly (e.g., Route 53) instead of relying on the default Gossip-based DNS.  



**Important:** Edit the cluster configuration if necessary, as some resources may exceed the AWS Free Tier limits.
```sh
kops edit cluster demok8scluster.k8s.local
```

### Step 3: Build the Cluster After creating the cluster definition, you need to apply the changes: (CHARGES WILL APPLY DONT USE ON FREE TIER)
```sh
kops update cluster demok8scluster.k8s.local --yes --state=s3://kops-your-storage-bucket
```
This process may take a few minutes.

### Step 4: Validate the Cluster Installation
```sh
kops validate cluster --state=s3://kops-your-storage-bucket
```

#### To See all cluster
```sh
aws s3 ls
```
To see all clusters managed by `kops`, use the following command:  

```sh
kops get cluster --state=s3://kops-abhi-storage
```

TO delete cluster
```sh
kops delete cluster --name=xxk8scluster.k8s.local --state=s3://kops-xxx-storage --yes
```
To delete an S3 bucket, follow these steps:  

### **1. Empty the Bucket** (Required before deletion)
```sh
aws s3 rm s3://kops-abhi-storage --recursive
```
This removes all objects in the bucket.

### **2. Delete the Bucket**
```sh
aws s3 rb s3://kops-abhi-storage --force
```
The `--force` flag ensures the bucket is removed after emptying.

#### **Alternative: If You Want to Manually Confirm Deletion**
Instead of `--force`, you can delete the bucket manually from the AWS Console:  
1. Go to **AWS S3 Console** → [S3 Buckets](https://s3.console.aws.amazon.com/s3).
2. Find **`kops-abhi-storage`**.
3. Empty the bucket → Delete the bucket.

-------------------------------------------------

### **Difference Between Kops, EKS, and Minikube**  

| Feature         | **Kops** (Kubernetes Operations) | **EKS** (Elastic Kubernetes Service) | **Minikube** |
|---------------|--------------------------------|--------------------------------|----------------|
| **Type** | Kubernetes cluster installer | Managed Kubernetes service | Local Kubernetes environment |
| **Best For** | Self-managed Kubernetes on AWS | Fully managed Kubernetes on AWS | Local development/testing |
| **Infrastructure** | AWS, GCP, OpenStack, Bare Metal | AWS-only | Local machine (VM or Docker) |
| **Management** | You manage the cluster | AWS manages the cluster | You manage the cluster |
| **Scalability** | Manually scalable | Auto-scaling by AWS | Limited (local only) |
| **Cost** | You pay for cloud resources | Pay-as-you-go (AWS charges) | Free (uses local resources) |
| **Networking** | Full control over networking | AWS manages networking | Local network only |
| **Production-Ready?** | ✅ Yes | ✅ Yes | ❌ No (for development only) |

---

### **1️⃣ What is Kops?**  
🔹 **Kops (Kubernetes Operations)** is a tool to **install, upgrade, and manage** Kubernetes clusters **on AWS, GCP, OpenStack, and bare metal**.  
🔹 **You control everything**: networking, security, updates, etc.  
🔹 You are responsible for managing the cluster.  

✅ **Use Kops if:**  
- You need a **self-managed Kubernetes cluster** on AWS.  
- You want full control over **networking, updates, and security**.  
- You don’t want to depend on **EKS (AWS Managed Kubernetes)**.  

---

### **2️⃣ What is EKS?**  
🔹 **EKS (Elastic Kubernetes Service)** is **AWS’s fully managed Kubernetes service**.  
🔹 AWS **creates, manages, and updates** the Kubernetes control plane.  
🔹 You **only manage worker nodes** (or use AWS Fargate for serverless nodes).  

✅ **Use EKS if:**  
- You want **AWS to handle Kubernetes management**.  
- You need **auto-scaling, security, and HA**.  
- You don’t want to **manually install and manage Kubernetes** like in Kops.  

---

### **3️⃣ What is Minikube?**  
🔹 **Minikube is a tool for running Kubernetes locally on your laptop**.  
🔹 Creates a **single-node Kubernetes cluster** using Virtual Machines (VM) or Docker.  
🔹 **Not for production** → It’s for **testing, learning, and development**.  

✅ **Use Minikube if:**  
- You want to **test Kubernetes locally**.  
- You are **developing a Kubernetes-based application**.  
- You don’t need a **real cloud environment** (AWS, GCP, etc.).  

---

### **4️⃣ Final Comparison**  
- **Kops →** Self-managed Kubernetes in AWS.  
- **EKS →** Fully managed Kubernetes in AWS.  
- **Minikube →** Local Kubernetes for development.  

**Which one should you use?**  
- **For production on AWS:** Use **EKS** (if you want managed) or **Kops** (if you want full control).  
- **For local development:** Use **Minikube**.  

Let me know if you need more details! 😊

### **Cost Difference: Kops vs. EKS**  

Yes, you're right! **With Kops, you only pay for the resources**, but **with EKS, you pay for both the resources and the EKS control plane**.

---

### **1️⃣ Kops (Kubernetes Operations) - Cost**  
- **Kops is free** (open-source tool).  
- You **only pay for the AWS resources** used:  
  ✅ **EC2 instances** (for control plane & worker nodes)  
  ✅ **EBS volumes** (for storage)  
  ✅ **ELB (Load Balancer)**  
  ✅ **S3** (for storing cluster state)  
  ✅ **Networking costs** (VPC, NAT, etc.)  

🔹 **No extra charge for Kubernetes control plane** (you run & manage it yourself).  
🔹 **More effort** required to manage, update, and secure the cluster.  

---

### **2️⃣ EKS (Elastic Kubernetes Service) - Cost**  
- **EKS charges you for both**:  
  ✅ **EKS control plane** → **$0.10 per hour per cluster** (~$72/month).  
  ✅ **AWS resources** (same as Kops): EC2, EBS, ELB, S3, networking, etc.  
- If you use **Fargate (serverless nodes)** instead of EC2, you **only pay for running pods** (no need to manage worker nodes).  

🔹 **Easier to manage** (AWS handles control plane, HA, and security).  
🔹 **More expensive** than Kops because of the control plane fee.  

---

### **3️⃣ Cost Comparison Example**  
| Feature  | **Kops (Self-Managed)** | **EKS (AWS Managed)** |
|----------|----------------|----------------|
| **Control Plane Cost** | **$0** (you manage it) | **$0.10/hour** ($72/month) |
| **Worker Nodes (EC2, EBS, etc.)** | ✅ Yes (same as EKS) | ✅ Yes (same as Kops) |
| **Load Balancers** | ✅ Yes | ✅ Yes |
| **S3 for State Storage** | ✅ Yes | ❌ No (AWS manages control plane) |
| **Networking (VPC, NAT, etc.)** | ✅ Yes | ✅ Yes |
| **Overall Cost** | **Cheaper, but more effort** | **More expensive, but easier to manage** |

---

### **4️⃣ Which One Should You Choose?**
- **Choose Kops** → If you want **full control and lower costs**, and are okay with managing everything.  
- **Choose EKS** → If you want **AWS to handle the Kubernetes control plane**, and are okay with paying extra for **convenience & managed services**.  

Let me know if you need more details! 🚀

# Kubernetes Setup and Running a Pod Using Minikube

## Prerequisites
Ensure you have the following before proceeding:
- A Linux or macOS system (Windows users can use WSL2 or Minikube)
- A user with sudo privileges
- Internet connection

---

## Step 1: Install Kubernetes (Minikube) and `kubectl`

### Install Minikube (For Local Kubernetes Cluster)
```sh
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

### Verify Installation
```sh
minikube version
```

### Install `kubectl` (Kubernetes CLI)
```sh
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

### Verify Installation
```sh
kubectl version --client
```

---

## Step 2: Start Kubernetes Cluster with Minikube
```sh
minikube start
```
**Explanation:** This command starts a local Kubernetes cluster using Minikube.

### Verify Cluster is Running
```sh
kubectl get nodes
```
**Explanation:** This checks if Kubernetes nodes are up and running.

---

## Step 3: Deploy a Pod
### Create a Simple Pod Manifest (YAML File)
Create a file named `pod.yaml` with the following content:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
    - name: nginx-container
      image: nginx:latest
      ports:
        - containerPort: 80
```

### Apply the Pod Configuration
```sh
kubectl apply -f pod.yaml
```
**Explanation:** This command deploys the pod to the Kubernetes cluster.

### Verify Pod is Running
```sh
kubectl get pods
```
**Explanation:** Lists all running pods and their statuses.

---

## Step 4: Access the Pod
### Get Pod Logs
```sh
kubectl logs my-pod
```
**Explanation:** Retrieves logs from the pod to debug and monitor application output.

### Execute a Command Inside the Pod
```sh
kubectl exec -it my-pod -- /bin/sh
```
**Explanation:** Opens a shell inside the running pod.

---

## Step 5: Clean Up
### Delete the Pod
```sh
kubectl delete pod my-pod
```
**Explanation:** Removes the deployed pod from the Kubernetes cluster.

### Stop Minikube (if needed)
```sh
minikube stop
```
**Explanation:** Stops the Minikube cluster.

### Delete Minikube Cluster (if needed)
```sh
minikube delete
```
**Explanation:** Completely removes the Minikube Kubernetes cluster from your system.

---

## Conclusion
You have successfully installed Kubernetes, set up `kubectl`, deployed a simple pod, and accessed it. Kubernetes is powerful for container orchestration, and this guide serves as a foundational setup.



