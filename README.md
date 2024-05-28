## Deploying a Three-tier Application on AWS EKS

### Overview

This Repository consists of the code to deploy a 3-tier Application (React, NodeJS, and MongoDB) on AWS EKS.

### Application Code 
The Application-Code directory contains the source code for the Three-Tier Web Application.

### Kubernetes-Manifests-file
This directory holds Kubernetes manifests for deploying the application on AWS EKS.

## Let's get started

### IAM Configuration
Create a user 3-tier-admin with Administrator Access.
```bash
AWS --> IAM --> Create user (3-tier-admin) --> Attach policies directly --> AdministratorAccess --> create admin
Open 3-tier-admin --> security credentials --> create Access keys --> cli --> create
```
You will have your Access key and Secret Access key.

### Setting up the workstation(EC2)
Create an EC2 Instance (ubuntu, t2.micro) in your prefered region and connect to it.
To get this repository, run the following command inside your git enabled terminal
```bash
$ git clone https://github.com/janvykumar/3-tier-AppDeploy-AWS-EKS.git
```

### Install Docker
```bash
$ sudo apt-get update
$ sudo apt install docker.io
$ sudo chown $USER /var/run/docker.sock (givinig perms for Ubuntu user to run docker)
$ docker ps
```

## Frontend

### Create Docker file
```bash
$ cd /home/ubuntu/3-tier-AppDeploy-AWS-EKS/Application-Code/frontend
```
Refer Application-Code/frontend/Dockerfile of this repo to get the file.
```bash
$ Run docker : docker build -t three-tier-frontend .
$ To see the latest image : docker images
$ To run the image : docker run -d -p 3000:3000 three-tier-frontend:latest ( react runs on port 3000 by default )
$ To see if the container is up and running : docker ps
```
You need to add an inbound rule for your instance allowing custom TCP 3000 port from anywhere.
Hit IPaddress:3000 on your browser --> you should be able to see the Todo app (frontend only) running.
After verification, you can kill this docker process : 
```bash
$ docker kill containerID
```

### Pushing the image to AWS ECR
Pre-req : Install AWS cli on your system
```bash
$ curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
$ sudo apt install unzip
$ unzip awscliv2.zip
$ sudo ./aws/install -i /usr/local/aws-cli -b /usr/local/bin --update
$ aws configure (provide your ID and secret key here, default region and output can be left blank)
```
Creating Repository on AWS ECR (Elastic Container Registry)
```bash
AWS --> ECR --> create repository --> public --> Name: three-tier-frontend --> create
```
We need to push our image in this repository. Click on "view push commands". You will see the commands here. Follow them.
```bash
Retrieve an authentication token and authenticate your Docker client to your registry. Use the AWS CLI:
$ aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws/d7p5s2q3
Build your Docker image using the following command ( This command needs to be run in the directory where your frontend docker file resides)
$ docker build -t three-tier-frontend .
$ docker tag three-tier-frontend:latest public.ecr.aws/d7p5s2q3/three-tier-frontend:latest
Run the following command to push this image to your newly created AWS repository:
$ docker push public.ecr.aws/d7p5s2q3/three-tier-frontend:latest
```

## Backend

Follow the same steps for backend as that of frontend. Create the Docker file for backend and push the image to ECR.
```bash
$ cd /home/ubuntu/3-tier-AppDeploy-AWS-EKS/Application-Code/backend
```
Create Docker file. Refer Application-Code/backend/Dockerfile of this repo to get the Docker file.

Creating ECR Repository and pushing the image
```bash
AWS --> ECR --> Create repository "three-tier-backend" and run the push commands.
In cli,
$ aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws/d7p5s2q3
$ docker build -t three-tier-backend .
$ docker run -d -p 8080:8080 three-tier-backend:latest (To run the image)
$ docker ps
Tag to ECR and push it
$ docker tag three-tier-backend:latest public.ecr.aws/d7p5s2q3/three-tier-backend:latest
$ docker push public.ecr.aws/d7p5s2q3/three-tier-backend:latest
```

## Set up MongoDB using kubernetes (EKS)

### Install kubectl
```bash
$ curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
$ chmod +x ./kubectl
$ sudo mv ./kubectl /usr/local/bin
$ kubectl version --short --client
```

### Install eksctl
```bash
$ curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
$ sudo mv /tmp/eksctl /usr/local/bin
$ eksctl version
```

### Setup EKS Cluster
```bash
$ eksctl create cluster --name three-tier-cluster --region us-west-2 --node-type t2.medium --nodes-min 2 --nodes-max 2 ( stack is deployed using cloud formation)
$ aws eks update-kubeconfig --region us-west-2 --name three-tier-cluster
$ kubectl get nodes
```

## Deploying MongoDB

```bash
$ cd /home/ubuntu/3-tier-AppDeploy-AWS-EKS/Kubernetes-Manifests-file/Database
```
Files in this folder are as follows:

1. deployment.yml - To create the pod

2. secrets.yml - To create username and password.

To encrypt the credentials
```bash
$ echo 'password_to_encrypt' | base64
```
              
To decrypt 
```bash
$ echo 'password_to_decrypt'| base64 --decode
```

Add the encrytpted username and password to secrets.yaml
              
3. services.yml - makes mongodb accessible to other services

To create a namespace
```bash
$ kubectl create namespace three-tier
```
To run these files
```bash
$ kubectl apply -f deployment.yaml
$ kubectl apply -f secrets.yaml
$ kubectl apply -f pv.yaml
$ kubectl apply -f pvc.yaml
$ kubectl apply -f sevice.yaml
```
```bash
To see the deployments --> kubectl get deployment -n three-tier
services running --> kubectl get svc -n three-tier
```
With this we have deployed MongoDB.



![Deployments and services running](https://github.com/janvykumar/3-tier-AppDeploy-AWS-EKS/blob/main/Screenshot%202024-05-28%20233214.png?raw=true)






