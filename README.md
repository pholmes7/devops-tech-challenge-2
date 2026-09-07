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

## Kubernetes Application Deployment

The containerized Flask application is deployed to Amazon EKS using Kubernetes. The Docker image is stored in Amazon Elastic Container Registry (ECR), allowing the EKS worker nodes to securely pull and run the application.

The initial Kubernetes deployment runs one application pod on the EKS managed node group. Kubernetes manages the application's desired state and automatically replaces the pod if it fails or is deleted.

### Kubernetes Architecture

The application currently follows this deployment flow:

```text
Local Flask Application
        ↓
Docker Image
        ↓
Amazon ECR
        ↓
Amazon EKS
        ↓
EKS Worker Node
        ↓
Kubernetes Deployment
        ↓
Flask Pod
        ↓
Kubernetes ClusterIP Service
```

Public access through an Application Load Balancer (ALB) will be configured in a later phase.

### Kubernetes Resources

The Kubernetes configuration is stored in:

```text
kubernetes/
├── deployment.yaml
└── service.yaml
```

The resources include:

- `deployment.yaml` - Defines and manages the Flask application pod.
- `service.yaml` - Creates a stable internal endpoint and routes traffic to the application pod.

### Application Deployment Configuration

The Kubernetes Deployment initially runs one replica of the Flask application.

The container uses the Docker image stored in Amazon ECR and listens on port `5000`.

Resource requests and limits are configured for the container:

```yaml
resources:
  requests:
    cpu: "25m"
    memory: "64Mi"
  limits:
    cpu: "250m"
    memory: "256Mi"
```

The CPU and memory requests establish the resource baseline for the pod and will later be used by the Horizontal Pod Autoscaler (HPA) when calculating CPU and memory utilization.

The Deployment also includes Kubernetes readiness and liveness probes against the application's `/` endpoint.

The readiness probe verifies that the application is ready to receive traffic, while the liveness probe allows Kubernetes to detect and restart an unhealthy container.

### Kubernetes Service

The application uses a `ClusterIP` Service:

```yaml
type: ClusterIP
```

The Service listens on port `80` and forwards traffic to the Flask application on port `5000`:

```text
Kubernetes Service :80
        ↓
Flask Pod :5000
```

The Service uses the application label to automatically locate the appropriate application pods:

```yaml
selector:
  app: tech-challenge-2-app
```

This provides a stable endpoint even when application pods are created, deleted, or replaced.

### Authenticate Docker to Amazon ECR

Before pushing an image, authenticate Docker with the Amazon ECR registry.

The standard AWS ECR authentication command is:

```bash
aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin <aws-account-id>.dkr.ecr.us-east-2.amazonaws.com
```

> **Windows PowerShell Note:** During development, PowerShell produced an HTTP 400 error when piping the ECR authentication token directly to Docker. Running the pipeline through `cmd.exe` resolved the issue:

```powershell
cmd.exe /c "aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin <aws-account-id>.dkr.ecr.us-east-2.amazonaws.com"
```

### Build and Push the Docker Image to ECR

Retrieve the ECR repository URL from Terraform:

```powershell
$ECR_REPO = terraform -chdir=terraform output -raw ecr_repository_url
```

Build the application image if needed:

```powershell
docker build -t tech-challenge-2-app ./app
```

Tag the local Docker image with the ECR repository:

```powershell
docker tag tech-challenge-2-app:latest ${ECR_REPO}:latest
```

Push the image to Amazon ECR:

```powershell
docker push ${ECR_REPO}:latest
```

Verify that the image exists in ECR:

```powershell
aws ecr describe-images --repository-name devops-tech-challenge-2-app --region us-east-2
```

### Configure kubectl for EKS

Configure the local Kubernetes configuration to communicate with the EKS cluster:

```powershell
aws eks update-kubeconfig --region us-east-2 --name tech-challenge-2-eks
```

Verify the active Kubernetes context:

```powershell
kubectl config current-context
```

Verify connectivity to the cluster:

```powershell
kubectl cluster-info
```

Verify that the EKS worker node is available:

```powershell
kubectl get nodes
```

The worker node should report a status of:

```text
Ready
```

### Validate the Kubernetes Configuration

Before deploying, the Kubernetes YAML files can be validated using a client-side dry run:

```powershell
kubectl apply --dry-run=client -f kubernetes\
```

The dry run validates the configuration without creating resources in the EKS cluster.

### Deploy the Application to EKS

Deploy the Kubernetes resources:

```powershell
kubectl apply -f kubernetes\
```

Verify the Deployment:

```powershell
kubectl get deployments
```

Verify the application pod:

```powershell
kubectl get pods
```

Verify the Service:

```powershell
kubectl get services
```

Verify that the Service has discovered the application pod:

```powershell
kubectl get endpoints tech-challenge-2-service
```

The initial application state should contain:

```text
EKS Worker Nodes:     1
Application Replicas: 1
Application Pods:     1
```

The application pod should report:

```text
READY:   1/1
STATUS:  Running
```

### Inspect the Application

View detailed information about the application pod:

```powershell
kubectl describe pod <pod-name>
```

View application logs:

```powershell
kubectl logs -l app=tech-challenge-2-app
```

The Flask application should report that it is listening on port `5000`.

### Test the Application

Because the application currently uses an internal `ClusterIP` Service, it can be tested locally using Kubernetes port forwarding.

Run:

```powershell
kubectl port-forward service/tech-challenge-2-service 8080:80
```

Then access:

```text
http://localhost:8080
```

The application should display:

```text
Hello, World!
```

The application can also be tested from another terminal:

```powershell
curl.exe http://localhost:8080
```

Expected response:

```text
Hello, World!
```

Stop the temporary port-forward with `Ctrl + C`.

### Kubernetes Self-Healing Test

Kubernetes self-healing was validated by manually deleting the running application pod.

First, identify the pod:

```powershell
kubectl get pods
```

Monitor the application pods:

```powershell
kubectl get pods -w
```

In another terminal, delete the running pod:

```powershell
kubectl delete pod <pod-name>
```

Because the Deployment specifies a desired state of one replica, Kubernetes automatically creates a replacement pod.

The expected behavior is:

```text
1 Running Pod
      ↓
Pod Deleted
      ↓
0 Running Pods
      ↓
Kubernetes Detects Desired-State Mismatch
      ↓
Replacement Pod Created
      ↓
1 Running Pod
```

Verify that the replacement pod reaches:

```text
READY:   1/1
STATUS:  Running
```

This validates Kubernetes' ability to maintain the application's desired state without manual intervention.

### Current Kubernetes Status

At the completion of this phase:

- The Flask application is containerized with Docker.
- The Docker image is stored in Amazon ECR.
- Amazon EKS is running with one `t3.small` worker node.
- The EKS managed node group can scale from 1 to 4 nodes.
- The application Deployment starts with one replica.
- CPU and memory resource requests and limits are configured.
- Readiness and liveness probes monitor application health.
- A ClusterIP Service routes traffic to the application pod.
- The application successfully returns `Hello, World!` through the Kubernetes Service.
- Kubernetes self-healing successfully replaces a deleted application pod.
- Public ALB access, Helm deployment, and autoscaling will be configured in later phases.

## Helm and Application Load Balancer

The Kubernetes application was packaged and deployed using Helm to provide reusable and configurable Kubernetes manifests. The Helm chart manages the application Deployment, ClusterIP Service, resource requests and limits, health probes, and Ingress configuration.

The AWS Load Balancer Controller was installed in the EKS cluster using Helm. OIDC and IAM resources were configured with Terraform to provide the controller with the AWS permissions required to manage load balancing resources.

A Kubernetes Ingress was configured with the `alb` IngressClass. The AWS Load Balancer Controller detected the Ingress and dynamically provisioned an internet-facing AWS Application Load Balancer.

Application traffic follows the path:

Internet → Application Load Balancer → Kubernetes Ingress → ClusterIP Service → Flask Pod

The ALB listens for HTTP traffic on port 80 and routes requests through the Kubernetes Service to the Flask application running on port 5000.

The deployment was validated by confirming the ALB was active and accessing the application through the public ALB DNS endpoint, which successfully returned:

`Hello, World!`

### AWS Load Balancer Controller

The controller uses a dedicated IAM role and Kubernetes ServiceAccount through IAM Roles for Service Accounts (IRSA). An EKS OIDC provider allows AWS to verify the controller's Kubernetes identity before granting the required AWS permissions.

During deployment, the controller initially entered a `CrashLoopBackOff` state because it could not automatically discover the VPC ID through EC2 instance metadata. The issue was identified using Kubernetes logs and resolved by retrieving the VPC ID from Terraform and explicitly providing it to the AWS Load Balancer Controller Helm release.

### Validation

The following were verified:

- Helm application release successfully deployed
- AWS Load Balancer Controller running successfully
- Kubernetes Ingress configured with the `alb` IngressClass
- Internet-facing Application Load Balancer successfully provisioned
- ALB registered with an AWS DNS endpoint
- Public traffic successfully routed to the Flask application
- Application returned `Hello, World!` through the ALB

## Autoscaling

The EKS environment is configured for both pod-level and node-level autoscaling.

### Horizontal Pod Autoscaler

Metrics Server was installed to provide CPU and memory utilization metrics to Kubernetes. The application HPA is configured to:

- Maintain a minimum of 1 application pod
- Scale up to 12 application pods
- Scale when CPU utilization reaches 50%
- Scale when memory utilization reaches 50%

Topology spread constraints were also added to distribute application replicas across available worker nodes.

### Cluster Autoscaler

Cluster Autoscaler was installed using Helm and configured to manage the EKS managed node group.

The node group uses `t3.small` instances and is configured with:

- Minimum nodes: 1
- Desired nodes: 1
- Maximum nodes: 4

A dedicated IAM role and IRSA configuration allow Cluster Autoscaler to securely manage the AWS Auto Scaling Group. Cluster Autoscaler successfully discovered the EKS node group and is running in the `kube-system` namespace.

### Validation

The autoscaling environment was validated before load testing:

- Metrics Server successfully reports CPU and memory usage
- HPA is active with CPU and memory targets of 50%
- Cluster Autoscaler is running successfully
- EKS currently maintains 1 worker node
- Application currently maintains 1 replica

Actual pod and node scaling behavior will be validated using Siege load testing.

## Load Testing and Autoscaling Validation

Siege was used to generate load against the application through the public AWS Application Load Balancer.

### Load Test

The test was executed with 25 concurrent users for 2 minutes.

```bash
siege -c 25 -t 2M http://<ALB-DNS-NAME>

## Jenkins Server Setup

A Jenkins server was provisioned on an AWS EC2 instance using Terraform to support CI/CD automation.

### Jenkins Configuration

- Provisioned a `t3.medium` EC2 instance for Jenkins.
- Installed Jenkins and Java.
- Installed and configured Git, Docker, AWS CLI, kubectl, and Helm.
- Configured the Jenkins user to run Docker commands.
- Created an EC2 IAM role and instance profile for secure AWS authentication without storing AWS access keys.
- Granted Jenkins access to Amazon ECR for container image operations.
- Configured EKS Access Entries to allow Jenkins to manage the Kubernetes cluster.
- Configured a GitHub Personal Access Token for access to the private repository.
- Verified Jenkins could successfully clone the private GitHub repository.
- Verified Jenkins could communicate with AWS, ECR, and EKS.

This configuration provides the foundation for the Jenkins CI/CD pipeline that will build the Docker image, push it to Amazon ECR, and deploy the application to Amazon EKS using Helm.

## Jenkins CI/CD Pipeline

Jenkins automates the application's build and deployment process using a pipeline defined in the `Jenkinsfile`.

The pipeline:

- Checks out the application from the private GitHub repository.
- Builds the Docker image.
- Authenticates with Amazon ECR.
- Tags and pushes the image to ECR.
- Connects to the Amazon EKS cluster.
- Deploys the new application version using Helm.
- Uses kubectl to verify the Kubernetes deployment.

Each Jenkins build uses a unique image tag based on the Jenkins build number, ensuring Kubernetes deploys the newly built application version.