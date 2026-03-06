# Terraform AWS EKS Infrastructure

This repository provisions a **complete AWS EKS infrastructure using Terraform**.  
It automates the setup of a Kubernetes environment on AWS, including networking, IAM roles, the EKS cluster, worker nodes, and deployment of **ArgoCD via Helm**.

This project demonstrates **Infrastructure as Code (IaC)** best practices and can serve as a foundation for **GitOps-based CI/CD pipelines**.

---

# Architecture Overview

The Terraform configuration provisions the following infrastructure:

AWS  
│  
├── VPC (10.0.0.0/16)  
│   ├── Public Subnet (10.0.1.0/24)  
│   └── Private Subnet (10.0.2.0/24)  
│  
├── IAM Roles  
│   ├── EKS Cluster Role  
│   └── Node Group Role  
│  
├── EKS Cluster (Kubernetes 1.28)  
│  
├── EKS Managed Node Group  
│   ├── Min: 1 node  
│   ├── Desired: 2 nodes  
│   └── Max: 4 nodes  
│  
├── ArgoCD Deployment (via Helm)  
│  
└── Optional S3 Bucket for Terraform State  

---

# Technologies Used

- Terraform
- AWS EKS
- AWS VPC
- AWS IAM
- Kubernetes Provider
- Helm Provider
- ArgoCD

---

# Project Structure


terraform-eks/
│
├── main.tf # Core infrastructure configuration
├── outputs.tf # Terraform outputs
└── README.md


---

# Prerequisites

Before running this project, ensure the following tools are installed:

- Terraform >= 1.5
- AWS CLI
- kubectl
- Helm
- AWS account with sufficient permissions

Verify installations:


terraform -v
aws --version
kubectl version --client
helm version


---

# Configure AWS Credentials

Configure AWS credentials using:


aws configure


Provide:


AWS Access Key
AWS Secret Key
Region: ap-south-1


---

# Terraform Providers Used

| Provider | Version |
|--------|--------|
| AWS | 6.34.0 |
| Kubernetes | >= 2.0.0 |
| Helm | >= 2.0.0 |

---

# Infrastructure Components

## 1. VPC

Creates a custom VPC:


CIDR: 10.0.0.0/16


Includes:

- Public subnet
- Private subnet
- DNS support enabled

---

## 2. IAM Roles

Two IAM roles are created.

### EKS Cluster Role

Used by the Kubernetes control plane.

Policy attached:


AmazonEKSClusterPolicy


---

### Node Group Role

Used by EC2 worker nodes.

Policies attached:


AmazonEKSWorkerNodePolicy
AmazonEKS_CNI_Policy
AmazonEC2ContainerRegistryReadOnly


---

## 3. EKS Cluster

Cluster configuration:


Name: my-eks-cluster
Version: 1.28
Region: ap-south-1


Subnets used:

- Public subnet
- Private subnet

---

## 4. Node Group

Managed node group configuration:


Desired Nodes: 2
Minimum Nodes: 1
Maximum Nodes: 4


Nodes are deployed inside the **private subnet**.

---

## 5. ArgoCD Deployment

ArgoCD is installed using the **Helm provider**.

Helm Chart:


Repository: https://argoproj.github.io/argo-helm

Chart: argo-cd
Version: 7.7.0
Namespace: argocd


The deployment waits until **EKS nodes are ready**.

---

# Terraform Backend

Currently using **local backend** for testing:


backend "local"


An **S3 bucket resource is included** for future remote state migration.

Example configuration if switching to S3:


backend "s3" {
bucket = "my-terraform-state"
key = "eks-terraform.tfstate"
region = "ap-south-1"
encrypt = true
}


---

# How to Use

## 1. Clone Repository


git clone https://github.com/YOUR_USERNAME/terraform-eks.git

cd terraform-eks


---

## 2. Initialize Terraform


terraform init


This downloads the required providers.

---

## 3. Validate Configuration


terraform validate


---

## 4. Preview Infrastructure


terraform plan


---

## 5. Deploy Infrastructure


terraform apply


Confirm when prompted:


yes


Terraform will provision:

- VPC
- Subnets
- IAM roles
- EKS cluster
- Node group
- ArgoCD

---

## 6. Configure kubectl

After deployment:


aws eks update-kubeconfig --region ap-south-1 --name my-eks-cluster


Verify nodes:


kubectl get nodes


---

# Access ArgoCD

Check ArgoCD pods:


kubectl get pods -n argocd


Port forward ArgoCD UI:


kubectl port-forward svc/argocd-server -n argocd 8080:443


Open in browser:


https://localhost:8080


---

# Terraform Outputs

After deployment Terraform will output:

### Cluster Endpoint


cluster_endpoint


### Cluster Name


cluster_name


View outputs:


terraform output


---

# Destroy Infrastructure

To remove all created resources:


terraform destroy


---

# Future Improvements

- Add **NAT Gateway for private subnet internet access**
- Use **S3 + DynamoDB for remote state locking**

---

# License

MIT License
