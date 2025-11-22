# Devops Project: video-converter

Converting mp4 videos to mp3 in a microservices architecture.

## Architecture

<p align="center">
  <img src="./Project documentation/ProjectArchitecture.png" width="600" title="Architecture" alt="Architecture">
</p>

## Deploying a Python-based Microservice Application on AWS EKS

### Introduction

This document provides a step-by-step guide for deploying a Python-based microservice application on AWS Elastic Kubernetes Service (EKS). The application comprises four major microservices: `auth-server`, `converter-module`, `database-server` (PostgreSQL and MongoDB), and `notification-server`.

### Prerequisites

Before you begin, ensure that the following prerequisites are met:

1. **Create an AWS Account:** If you do not have an AWS account, create one by following the steps.
2. **Install Helm:** Install Helm using official instructions.
3. **Python:** Install Python from the official website.
4. **AWS CLI:** Install AWS CLI.
5. **kubectl:** Install the latest version of kubectl.
6. **Databases:** PostgreSQL & MongoDB will be installed via Helm on EKS.

### High Level Flow of Application Deployment

1. Install PostgreSQL and MongoDB using Helm charts.
2. Install RabbitMQ and create queues `mp3` and `video`.
3. Deploy microservices (`auth`, `gateway`, `converter`, `notification`).
4. Validate deployment using:

   ```bash
   kubectl get all
   ```
5. Destroy EKS resources after completion.

---

## Low Level Steps

### Cluster Creation

#### 1. Log in to AWS Console

Use your AWS credentials.

#### 2. Create eksCluster IAM Role

Attach required policies including `AmazonEKS_CNI_Policy`.

<p align="center">
  <img src="./Project documentation/ekscluster_role.png" width="600">
</p>

#### 3. Create Node Role - AmazonEKSNodeRole

Attach:

* AmazonEKS_CNI_Policy
* AmazonEBSCSIDriverPolicy
* AmazonEC2ContainerRegistryReadOnly

<p align="center">
  <img src="./Project documentation/node_iam.png" width="600">
</p>

#### 4. Create EKS Cluster

Configure VPC, Subnets, IAM role.

#### 5. Create Node Group

Select AMI (default), instance type (t3.medium recommended).

#### 6. Add Inbound Rules to Node Security Group

Ensure ports:

* 30003 (PostgreSQL)
* 30005 (MongoDB)
* 30004 (RabbitMQ)
* 30002 (Gateway/Auth)

<p align="center">
  <img src="./Project documentation/inbound_rules_sg.png" width="600">
</p>

#### 7. Enable EBS CSI Addon

<p align="center">
  <img src="./Project documentation/ebs_addon.png" width="600">
</p>

---

## Deploying the Application on EKS

### Set kubeconfig

```bash
aws eks update-kubeconfig --name <cluster_name> --region <aws_region>
```

---

# Commands

## MongoDB Installation

```bash
cd Helm_charts/MongoDB
helm install mongo .
```

### Connect to MongoDB

```bash
mongosh "mongodb://<username>:<password>@<nodeIP>:30005/mp3s?authSource=admin"
```

## PostgreSQL Installation

```bash
cd ../Postgres
helm install postgres .
```

### Connect to PostgreSQL

```bash
psql "postgres://<username>:<password>@<nodeIP>:30003/authdb"
```

## RabbitMQ Installation

```bash
helm install rabbitmq .
```

Access UI:

```
http://<nodeIP>:30004
User: guest
Pass: guest
```

Create queues: `mp3`, `video`

---

## Deploy Microservices

### Auth Service

```bash
cd auth-service/manifest
kubectl apply -f .
```

### Gateway Service

```bash
dcd gateway-service/manifest
kubectl apply -f .
```

### Converter Service

Update email/password in `converter/manifest/secret.yaml`.

```bash
cd converter-service/manifest
kubectl apply -f .
```

### Notification Service

```bash
cd notification-service/manifest
kubectl apply -f .
```

---

## Validate Deployment

```bash
kubectl get all
```

---

## Configure Gmail App Password

Enable 2FA → App passwords → Generate password → Add to `notification-service/manifest/secret.yaml`.

---

# API Endpoints

## Login

```bash
curl -X POST http://<nodeIP>:30002/login -u <email>:<password>
```

## Upload Video

```bash
curl -X POST -F "file=@video.mp4" -H "Authorization: Bearer <JWT_TOKEN>" http://<nodeIP>:30002/upload
```

## Download Converted MP3

```bash
curl --output output.mp3 -X GET -H "Authorization: Bearer <JWT_TOKEN>" "http://<nodeIP>:30002/download?fid=<file_id>"
```

---

# Destroying the Infrastructure

### 1. Delete Node Group

Delete from AWS Console → EKS → Compute → Node Groups.

### 2. Delete Cluster

Once node group deleted:

```bash
aws eks delete-cluster --name <cluster_name> --region <aws_region>
```
