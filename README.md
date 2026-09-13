# StreamingApp – Orchestration and Scaling Project

## Overview

This project demonstrates the deployment, orchestration, monitoring, and scaling of a MERN-based Streaming Application using AWS cloud services, Docker, Jenkins, Kubernetes (EKS), Helm, and CloudWatch.

The objective of this project is to implement a complete CI/CD pipeline that automates application build, containerization, image storage, deployment, monitoring, and notification workflows.

---

## Technologies Used

| Category           | Technology                                    |
| ------------------ | --------------------------------------------- |
| Source Control     | Git, GitHub                                   |
| CI/CD              | Jenkins                                       |
| Containerization   | Docker                                        |
| Container Registry | Amazon ECR                                    |
| Orchestration      | Amazon EKS                                    |
| Package Management | Helm                                          |
| Monitoring         | Amazon CloudWatch                             |
| Notifications      | Amazon SNS                                    |
| Cloud Provider     | AWS                                           |
| Application Stack  | MERN (MongoDB, Express.js, React.js, Node.js) |

---

## Project Repository

**Source Repository**

[https://github.com/UnpredictablePrashant/StreamingApp.git](https://github.com/UnpredictablePrashant/StreamingApp.git)

<img width="761" height="428" alt="image" src="https://github.com/user-attachments/assets/da37a197-c127-4694-af0b-cff29aafb038" />


---

## Solution Architecture

```text
GitHub Repository
        │
        ▼
     Jenkins
        │
        ├── Build Frontend Image
        ├── Build Backend Image
        │
        ▼
   Amazon ECR
        │
        ▼
 Amazon EKS Cluster
        │
        ▼
      Helm
        │
        ▼
Frontend Pods + Backend Pods
        │
        ▼
 Application Services
        │
        ▼
 Amazon CloudWatch
        │
        ▼
 Amazon SNS Email Alerts
```

---

## Project Workflow

### Step 1: Version Control

* Forked the project repository.
* Maintained source code using Git and GitHub.
* Pulled updates from the upstream repository when required.

### Step 2: Application Containerization

Created Dockerfiles for:

* Frontend Application
* Backend Application

Built Docker images for both services.

Example:

```bash
docker build -t frontend .
docker build -t backend .
```

Dockerfiles Created
•	frontend/Dockerfile 
•	backend/authService/Dockerfile 
•	backend/streamingService/Dockerfile
•	backend/adminService/Dockerfile
•	backend/chatService/Dockerfile

Local Testing with Docker Compose:
1. Built and started all services:
docker-compose up --build

2. Verified all 5 services running on their respective ports:

•	Frontend

•	Auth Service: http://localhost:3001

•	Streaming Service: http://localhost:3002

•	Admin Service: http://localhost:3003

•	Chat Service: http://localhost:3004

3. Committed changes to GitHub:


```bash
git add .

git commit -m "Dokcer-compse file created"

git push origin main

```
---

### Step 3: Amazon ECR

Created Amazon Elastic Container Registry repositories.

Tasks performed:

* Created separate ECR repositories.
* Authenticated Docker with ECR.
* Tagged Docker images.
* Pushed images to ECR.

<img width="1587" height="385" alt="image" src="https://github.com/user-attachments/assets/d96f9347-22f6-4566-bc10-25bb761e44d8" />

Example:

```bash
aws ecr get-login-password --region <region> \
| docker login --username AWS --password-stdin <account-id>.dkr.ecr.<region>.amazonaws.com

dokcer tag app:v1 <account-id>.dkr.ecr.<region>.amazonaws.com/app:v1

docker push <ecr-image-uri>
```

<img width="965" height="382" alt="Screenshot 2026-06-07 215530" src="https://github.com/user-attachments/assets/feed3c1e-5f44-44f0-8261-159b0c93642b" />

<img width="1132" height="247" alt="Screenshot 2026-06-07 215500" src="https://github.com/user-attachments/assets/2d2bdbfe-6210-4e9a-8986-fd15039320ec" />



---

### Step 4: Jenkins CI/CD Pipeline

Jenkins was installed on an AWS EC2 instance.

#### Jenkins Responsibilities

* Pull source code from GitHub
* Build Docker images
* Authenticate with ECR
* Push Docker images to ECR
* Deploy updated images to EKS using Helm

#### Pipeline Stages

1. Checkout Source Code
2. Build and Push Frontend — Builds and pushes frontend image
3. Login to ECR — Authenticates Docker with Amazon ECR
4. Build and Push Frontend — Builds and pushes frontend image
5. Build and Push Auth — Builds and pushes auth service image
6. Build and Push Streaming — Builds and pushes streaming service image
7. Build and Push Admin — Builds and pushes admin service image
8. Build and Push Chat — Builds and pushes chat service image



<img width="940" height="449" alt="image" src="https://github.com/user-attachments/assets/da49f1db-8c1c-4d48-9c22-b86808adc209" />
<img width="940" height="478" alt="image" src="https://github.com/user-attachments/assets/6194653c-05b2-401e-9c1c-5286c6e2ba14" />


---

#### GitHub Webhook — Auto-trigger CI/CD
1. Configured webhook in GitHub repository Settings → Webhooks:

   ◦	Payload URL: http://<EC2-IP>:8080/github-webhook/

   ◦	Content type: application/json

   ◦	Event: Push events

2. Enabled 'GitHub hook trigger for GITScm polling' in Jenkins pipeline configuration
3. Verified: pushing a commit automatically triggers Jenkins pipeline build

# Kubernetes Deployment using Amazon EKS and Helm

## Overview

Amazon Elastic Kubernetes Service (EKS) was used to deploy and manage the containerized Streaming Application. 
Helm was utilized to package and deploy all application microservices through a single reusable chart, simplifying deployment and management.

---

## Tools Installed on EC2

### Install eksctl

```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp

sudo mv /tmp/eksctl /usr/local/bin
```

### Install kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

### Install Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

---

## EKS Cluster Creation

### Create Cluster

```bash
eksctl create cluster \
  --name streaming-cluster \
  --region ap-south-1 \
  --nodegroup-name streaming-nodes \
  --node-type t3.small \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 2 \
  --managed
```

### Cluster Configuration

| Parameter          | Value               |
| ------------------ | ------------------- |
| Cluster Name       | streaming-cluster   |
| Region             | ap-south-1          |
| Node Group         | streaming-nodes     |
| Node Type          | t3.small            |
| Nodes              | 2                   |
| Kubernetes Version | v1.34.7-eks-7fcd7ec |

### Configure kubectl

```bash
aws eks update-kubeconfig --region ap-south-1 --name streaming-cluster
```

### Verify Nodes

```bash
kubectl get nodes
```

---

## Helm Chart Structure

A custom Helm chart was created to deploy all application services.

```text
streaming-app/
├── Chart.yaml
├── values.yaml
└── templates/
    ├── deployment.yaml
    └── service.yaml
```

### Components

* **Chart.yaml** – Chart metadata including name, version, and description.
* **values.yaml** – Configurable values such as image repositories, tags, and replica counts.
* **deployment.yaml** – Kubernetes Deployments for all five services.
* **service.yaml** – Kubernetes Services configuration.

---

## Helm Deployment

### Deploy Application

```bash
helm install streaming-app ~/streaming-app
```

### Verify Pods

```bash
kubectl get pods
```

All application pods were successfully deployed and reached the **Running** state:

* admin
* auth
* chat
* frontend
* streaming

<img width="1175" height="536" alt="image" src="https://github.com/user-attachments/assets/c0e69169-ae47-4ed3-a027-b35c6ae3a5dc" />

---

## Monitoring and Logging

# Monitoring and Logging with Amazon CloudWatch

## Overview

Amazon CloudWatch was configured to provide monitoring and observability for the Amazon EKS cluster. The CloudWatch Observability Addon was installed to collect metrics and logs from Kubernetes workloads. Additionally, a CloudWatch alarm was created to monitor CPU utilization and notify administrators when resource usage exceeds defined thresholds.

---

## CloudWatch Observability Addon

### Install CloudWatch Observability Addon

```bash
aws eks create-addon \
  --cluster-name streaming-cluster \
  --addon-name amazon-cloudwatch-observability \
  --region ap-south-1
```

### Addon Details

| Parameter  | Value                           |
| ---------- | ------------------------------- |
| Addon Name | amazon-cloudwatch-observability |
| Version    | v5.4.0-eksbuild.1               |
| Cluster    | streaming-cluster               |
| Namespace  | amazon-cloudwatch               |

### Purpose

The CloudWatch Observability Addon enables:

* Collection of container and pod metrics
* Kubernetes workload monitoring
* Centralized log aggregation
* Enhanced observability for Amazon EKS workloads

---

## CloudWatch Alarm Configuration

To monitor cluster resource utilization, a CloudWatch alarm was created for CPU usage.

### Create CPU Utilization Alarm

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name "EKS-CPU-High" \
  --alarm-description "EKS CPU utilization high" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --region ap-south-1
```
<img width="940" height="404" alt="image" src="https://github.com/user-attachments/assets/ea197e8b-f5e5-4bd1-8be2-94f3faf06a0c" />

### Alarm Details

| Parameter          | Value                   |
| ------------------ | ----------------------- |
| Alarm Name         | EKS-CPU-High            |
| Metric             | CPUUtilization          |
| Namespace          | AWS/EC2                 |
| Threshold          | Greater than 80%        |
| Evaluation Periods | 2                       |
| Period             | 300 seconds (5 minutes) |
| Region             | ap-south-1              |

### Alert Condition

The alarm transitions to the **ALARM** state when:

* CPU utilization exceeds **80%**
* For **two consecutive evaluation periods**
* Each evaluation period is **5 minutes**

---

## Verification

The CloudWatch alarm was successfully created and verified in the AWS CloudWatch Console.
<img width="940" height="411" alt="image" src="https://github.com/user-attachments/assets/fec88619-26bc-4799-a4e9-51a8ddf49234" />

---

## CI/CD Workflow

```text
Developer Push
       │
       ▼
    GitHub
       │
       ▼
    Jenkins
       │
       ▼
 Docker Build
       │
       ▼
 Amazon ECR
       │
       ▼
 Amazon EKS
       │
       ▼
     Helm
       │
       ▼
 CloudWatch

```
