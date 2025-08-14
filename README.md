# Provisioning Terraform EKS Cluster

This project demonstrates how I provisioned a **production-ready Amazon EKS cluster** using **Terraform**.  
It uses the **prebuilt [`terraform-aws-modules/eks/aws`](https://registry.terraform.io/modules/terraform-aws-modules/eks/aws/latest)** module for creating the EKS cluster, along with a custom VPC configuration to provide the networking infrastructure.

---

## 📂 Project Structure


├── eks.tf                 # EKS module configuration
├── provider.tf            # AWS provider setup
├── terraform.tf           # Terraform backend & settings
├── variables.tf           # Variables for reusable config
├── vpc.tf                 # VPC networking configuration

---

## ⚙️ How It Works

### **1. Provider Configuration**
In `provider.tf`, we define the AWS provider and region using **local variables**.

```hcl
provider "aws" {
  region = local.region
}
````

---

### **2. Local Variables**

I used `locals` in `eks.tf` to make the configuration reusable and easy to modify for different environments.

```hcl
locals {
  region          = "us-east-2"
  name            = "terraform-eks"
  vpc_cidr        = "10.0.0.0/16"
  azs             = ["us-east-2a", "us-east-2b"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24"]
  intra_subnets   = ["10.0.5.0/24", "10.0.6.0/24"]
  env             = "dev"
}
```

---

### **3. VPC Networking**

The `vpc.tf` file provisions a custom VPC with:

* Private subnets for worker nodes
* Public subnets for internet-facing resources
* Intra subnets for the control plane

This ensures **isolation and security** for the EKS control plane and workloads.

---

### **4. EKS Cluster Configuration**

In `eks.tf`,  import the **EKS Terraform module** and configure:

* **Cluster name & version** (`1.33`)
* **Public endpoint access**
* **Networking** (VPC ID & subnets from `module.vpc`)
* **Add-ons**: CoreDNS, kube-proxy, VPC CNI
* **Control plane subnet** (from intra subnets)
* **Managed Node Group**:

  * Instance type: `t3.medium`
  * Scaling: min=2, max=3, desired=2
  * Capacity type: SPOT for cost optimization

Example from `eks.tf`:

```hcl
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "~> 21.0"

  name                  = local.name
  kubernetes_version    = "1.33"
  endpoint_public_access = true

  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets

  addons = {
    coredns   = { most-recent = true }
    kube-proxy = { most-recent = true }
    vpc-cni   = { most-recent = true }
  }

  control_plane_subnet_ids = module.vpc.intra_subnets

  eks_managed_node_groups = {
    eks-cluster-ng = {
      instance_types = ["t3.medium"]
      min_size       = 2
      max_size       = 3
      desired_size   = 2
      capacity_type  = "SPOT"
    }
  }

  tags = {
    Environment = local.env
    Terraform   = "true"
  }
}
```

---

## 🚀 Deployment Steps

1. **Initialize Terraform**

   ```bash
   terraform init
   ```

2. **Validate Configuration**

   ```bash
   terraform validate
   ```

3. **Plan the Deployment**

   ```bash
   terraform plan
   ```

4. **Apply Changes**

   ```bash
   terraform apply
   ```

---

## ✅ Features

* Uses **official EKS Terraform module** for stability & best practices
* Custom VPC with private, public, and intra subnets
* Configurable via **local variables** for multiple environments
* Adds **EKS-managed node groups** with scaling & SPOT instance support
* Automatically installs **CoreDNS, kube-proxy, and VPC CNI add-ons**

---


## 📜 License

This project is for **educational purposes only** and follows the usage guidelines of the Terraform AWS modules repository.