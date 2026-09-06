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