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

## Let's start with deploying frontend first. We will be installing other tools as and when required.

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
docker kill containerID
```







