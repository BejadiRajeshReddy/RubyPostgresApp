# Todo-App Using Ruby Rails PostgreSQL

This project demonstrates a complete DevOps pipeline using a sample Ruby on Rails application deployed with PostgreSQL. It covers Dockerization, Kubernetes deployment, GitOps with ArgoCD, and CI/CD automation using Tekton.


## 📌 Project Overview

This project is a part of a DevOps assessment designed to evaluate hands-on knowledge in:

- Docker
- Kubernetes
- ArgoCD (GitOps)
- Tekton (CI/CD)

A Ruby on Rails app is containerized, deployed on Kubernetes using Helm or YAML, configured for GitOps with ArgoCD, and automated using Tekton pipelines.



## 🧰 Tech Stack

- **Backend**: Ruby on Rails
- **Database**: PostgreSQL
- **Containerization**: Docker
- **Orchestration**: Kubernetes (Minikube)
- **GitOps**: ArgoCD
- **CI/CD**: Tekton Pipelines & Tekton Dashboard

---

## 🚀 Features

- Dockerized Rails and PostgreSQL setup using separate containers.
- Kubernetes manifests for app deployment and StatefulSet for PostgreSQL.
- ArgoCD setup for continuous delivery through GitOps.
- Tekton pipeline to automate build and image push to Docker Hub.

---

## 🏗️ Project Structure

```
RAILS-POSTGRES-APP/
│
├── app/                    # Rails application source code
├── config/                 # App configuration files
├── db/                     # Database seeds and migrations
├── kubernetes-files/       # Kubernetes manifests (deployments, services, ingress)
├── tekton/                 # Tekton pipeline and task definitions
├── Dockerfile              # Dockerfile for Rails app
├── docker-compose.yaml     # Docker Compose setup for local development
├── init.sql                # DB initialization script
├── Gemfile                 # Gem dependencies
├── README.md               # You're reading it!
└── ...

```


# Installation

Step‑by‑step walkthrough to get **all** of the required tools installed on a fresh Ubuntu 22.04 LTS (Linux) workstation.


## 🐧 Step 1: Install Ubuntu via Microsoft Store

### ✅ Requirements:
- Windows 10 version 2004 or higher (Build 19041+) OR Windows 11
- WSL enabled



### 🛍️ Install Ubuntu from Microsoft Store

- Open **Microsoft Store** 

- Search for **"Ubuntu"**
- Choose **Ubuntu 22.04 LTS** (or your preferred version)
- Click **Install**
- Once installed, **open Ubuntu** from Start Menu
- It will initialize – enter a **username and password** when prompted.

---

## 🐳 Step 2: Install Docker Desktop

### ✅ Requirements:
- WSL 2 backend enabled
- Ubuntu installed as default WSL distro

### 🔽 Download & Install Docker Desktop

- Go to: [https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)
- Download the Windows installer
- Run the installer
- During setup, ensure **“Use WSL 2 based engine”** is checked
- After installation, restart your PC



---


## 1. Update your system

Always start by updating package lists and upgrading any existing packages:

```bash
sudo apt update && sudo apt upgrade -y
```

---

## 2. Install Git

You’ll need Git to clone repositories, push manifests, etc.

```bash
sudo apt install -y git
git --version         # verify
```

---

## 3. Install Docker & Docker Compose


### a. Add Docker’s official GPG key and repository

```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
   https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### b. Install Docker Engine and Compose plugin

```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### c. Post‑install steps

```bash
# Run docker commands without sudo:
sudo usermod -aG docker $USER
# (Log out / back in for group change to take effect)
docker version
docker compose version
```

---

## 4. Install Ruby & Rails

We’ll use the Ubuntu packages for simplicity:

```bash
sudo apt install -y \
  ruby-full \
  build-essential \
  zlib1g-dev \
  libpq-dev

# Install the Rails gem (v6.x):
sudo gem install rails -v '~>6.0'
rails --version    # should show Rails 6.x
```

---

## 5. Install PostgreSQL Client

Your Rails app will talk to Postgres; you need the client libraries and `psql` tool:

```bash
sudo apt install -y postgresql-client
psql --version
```

---

## 6. Install kubectl (Kubernetes CLI)

```bash
curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg \
  | sudo gpg --dearmor -o /usr/share/keyrings/k8s-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/k8s-archive-keyring.gpg] \
  https://apt.kubernetes.io/ kubernetes-xenial main" \
  | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt update
sudo apt install -y kubectl
kubectl version --client
```

---

## 7. Install a Local Kubernetes Cluster

### Minikube

```bash
# Install dependencies:
sudo apt install -y conntrack

# Download and install:
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Start cluster:
minikube start --driver=docker
```

---

## 8. (Optional) Install Helm

Helm makes it easy to install ingress‑nginx or other charts:

```bash
curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
sudo apt install -y apt-transport-https --no-install-recommends
echo "deb https://baltocdn.com/helm/stable/debian/ all main" \
  | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt update
sudo apt install -y helm
helm version
```

---

## 9. Install Argo CD CLI

```bash
# Download binary
curl -sSL -o argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
chmod +x argocd
sudo mv argocd /usr/local/bin/argocd
argocd version --client
```

---

## 10. Install Tekton Pipelines & Dashboard

> We won’t install a CLI for Tekton; you’ll manage it via `kubectl` and the Web UI.

```bash
# Tekton Pipelines CRDs & controller:
kubectl apply --filename https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml

# Tekton Dashboard:
kubectl apply --filename https://storage.googleapis.com/tekton-releases/dashboard/latest/tekton-dashboard-release.yaml

# Expose Dashboard locally:
kubectl port-forward svc/tekton-dashboard -n tekton-pipelines 9097:8080
```

Visit <http://localhost:9097> in your browser to see the Tekton Dashboard.

---
---

# 🔧 Setup Instructions

### 1. Clone the Repository

```bash
git clone https://github.com/BejadiRajeshReddy/RubyPostgresApp
cd RubyPostgresApp
```

### 2. Docker

- Build and run the Rails app and PostgreSQL in separate containers using Docker Compose (or individual Dockerfiles).

```bash
docker-compose up --build
```

### 3. Kubernetes (Minikube)

- Start Minikube

```bash
minikube start
```

- Apply Kubernetes manifests

```bash
kubectl apply -f kubernetes-files/dbdeploy.yaml
kubectl apply -f kubernetes-files/dbservice.yaml
kubectl apply -f kubernetes-files/ingress.yaml
kubectl apply -f kubernetes-files/webdeploy.yaml
kubectl apply -f kubernetes-files/webservice.yaml
```
- Run The App
```bash
kubectl port-forward service/webapp-service 3000:3000
```


### 4. ArgoCD (GitOps)

- Install ArgoCD:
  [Install ArgoCD](https://argo-cd.readthedocs.io/en/stable/getting_started/)

- Port forward the UI and login:
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

- Check all pods and service

```bash
kubectl get all -n argocd
kubectl get svc -n argocd
minikube service list -n argocd
```


- Start the ArgoCD UI
```bash
minikube service argocd-server -n argocd
```

### 5. Tekton (CI/CD)

- Install Tekton and Tekton Dashboard:
  [Install Tekton](https://tekton.dev/docs/pipelines/installation/)


- Check pipelines
    ```bash
    kubectl get pods -n tekton-pipelines
    kubectl get svc tekton-dashboard -n tekton-pipelines
    ```

- Monitor running Status

```bash
kubectl get pods --namespace tekton-pipelines --watch
```
- Access The Dashboard
```bash
kubectl port-forward svc/tekton-dashboard -n tekton-pipelines 9097:9097
```

- Run Pipeline

```bash
kubectl apply -f docker-credentials.yaml
kubectl apply -f git-clone-task.yaml
kubectl apply -f kaniko-task.yaml
kubectl apply -f pipeline.yaml
kubectl apply -f service-account.yaml
```

- Create Pipeline Run
```bash
kubectl create -f pipelinerun.yaml
```

- Use the Tekton Dashboard to manually run the pipeline.

---

## 🔑 Environment Variables

Set the following in your `.env` or deployment secrets:

- `POSTGRES_DB`
- `POSTGRES_USER`
- `POSTGRES_PASSWORD`
- `RAILS_MASTER_KEY`

---

## 🐳 Docker Hub

```
https://hub.docker.com/u/rajeshreddy0
```
