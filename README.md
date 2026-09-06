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