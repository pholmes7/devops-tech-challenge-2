# DevOps Technical Challenge 2

## Project Overview

This project demonstrates the deployment of a containerized web application to Amazon EKS using Kubernetes, Terraform, Docker, Helm, and Jenkins.

The application is a simple Python Flask web application that displays:

**Hello, World!**

The project will demonstrate Infrastructure as Code, container orchestration, application and infrastructure auto scaling, load testing, and automated CI/CD deployment.

## Technologies

- Python
- Flask
- Docker
- Amazon ECR
- Amazon EKS
- Kubernetes
- Terraform
- Helm
- Jenkins
- AWS Application Load Balancer
- Siege

## Local Application Setup

### Requirements

- Python 3
- pip

### Install Dependencies

From the project root:

```bash
pip install -r app/requirements.txt
```

### Run the Application

From the project root:

```bash
python app/app.py
```

Open a browser and navigate to:

`http://localhost:5000`

The application should display:

**Hello, World!**
## Docker Setup

The Flask application is containerized using Docker. The Dockerfile and application dependencies are located in the `app` directory.

### Build the Docker Image

From the project root, run:

```bash
docker build -t tech-challenge-2-app ./app
```

This builds the Docker image using the Dockerfile located in the `app` directory.

### Run the Docker Container

Run the container and map port 5000 on the local machine to port 5000 inside the container:

```bash
docker run -d --name tech-challenge-2-container -p 5000:5000 tech-challenge-2-app
```

### Access the Application

Open a browser and navigate to:

`http://localhost:5000`

The application should display:

**Hello, World!**

### Verify the Container

To verify that the container is running:

```bash
docker ps
```

To view the application logs:

```bash
docker logs tech-challenge-2-container
```

### Stop and Restart the Container

Stop the container:

```bash
docker stop tech-challenge-2-container
```

Restart the existing container:

```bash
docker start tech-challenge-2-container
```
---

## AWS Infrastructure with Terraform

Terraform is used to provision the AWS infrastructure required to run the application on Amazon EKS.

### Infrastructure Provisioned

The Terraform configuration creates:

- Custom VPC
- Two public subnets across two Availability Zones
- Two private subnets across two Availability Zones
- Internet Gateway
- NAT Gateway
- Public and private route tables
- IAM roles for the EKS cluster and worker nodes
- Amazon ECR repository
- Amazon EKS cluster
- EKS managed node group

The EKS worker nodes run in private subnets while the public subnets provide the networking required for internet-facing resources such as the Application Load Balancer.

### EKS Node Configuration

The managed node group is configured with:

- Instance type: `t3.small`
- Minimum nodes: `1`
- Desired nodes: `1`
- Maximum nodes: `4`

The initial environment therefore starts with one worker node and can later scale up to four nodes.

### Terraform Directory

Terraform configuration files are located in:

```text
terraform/
├── providers.tf
├── variables.tf
├── terraform.tfvars
├── networking.tf
├── iam.tf
├── ecr.tf
├── eks.tf
└── outputs.tf
```

### Deploying the Infrastructure

Navigate to the Terraform directory:

```bash
cd terraform
```

Initialize Terraform:

```bash
terraform init
```

Format and validate the configuration:

```bash
terraform fmt
terraform validate
```

Preview the infrastructure changes:

```bash
terraform plan
```

Deploy the infrastructure:

```bash
terraform apply
```

Review the Terraform plan and enter `yes` when prompted to approve the deployment.

### Terraform Outputs

After deployment, important infrastructure information can be displayed with:

```bash
terraform output
```

Outputs include:

- AWS region
- VPC ID
- Public subnet IDs
- Private subnet IDs
- EKS cluster name
- EKS API endpoint
- EKS managed node group name
- ECR repository URL

For example, the ECR repository URL can be retrieved with:

```bash
terraform output -raw ecr_repository_url
```

### Configure kubectl for EKS

After the EKS cluster has been created, configure the local Kubernetes client:

```bash
aws eks update-kubeconfig --region us-east-2 --name tech-challenge-2-eks
```

Verify the connection:

```bash
kubectl cluster-info
```

Verify the worker nodes:

```bash
kubectl get nodes
```

The initial deployment should show one `t3.small` worker node in the `Ready` state.

Core EKS system workloads can be verified with:

```bash
kubectl get pods -n kube-system
```