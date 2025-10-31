#  DevOps Assessment – Next.js Application Deployment

### Author: Sai Adithya Yarramasu  
### Repository: [adithya160-gif/devops-assessment-next](https://github.com/adithya160-gif/devops-assessment-next)

---

## Project Overview

This project is part of a **DevOps Internship Assessment**, designed to demonstrate practical skills in:

- Application containerization using **Docker**
- Automating CI/CD pipelines using **GitHub Actions**
- Hosting container images on **GitHub Container Registry (GHCR)**
- Deploying the containerized application on **Kubernetes (Minikube)**

The application used in this project is a **Next.js web app**.

---

##  Tech Stack

| Category | Tools Used |
|-----------|-------------|
| Application Framework | Next.js (Node.js 20) |
| Version Control | Git, GitHub |
| CI/CD | GitHub Actions |
| Containerization | Docker |
| Container Registry | GitHub Container Registry (GHCR) |
| Orchestration | Kubernetes (Minikube) |
| OS/Environment | macOS / Docker Desktop |

---

## Project Setup (Local Development)

### 🔹Clone the Repository
bash
git clone https://github.com/adithya160-gif/devops-assessment-next.git
cd devops-assessment-next

1.Install Dependencies
 npm install

 Run the Application Locally
 npm run dev

2.Dockerization
  The project includes a multi-stage Dockerfile for efficient production builds.

  Build Docker Image
  docker build -t ghcr.io/adithya160-gif/devops-assessment-next:local .

  Run Container Locally
  docker run -p 3000:3000 ghcr.io/adithya160-gif/devops-assessment-next:local

3.GitHub Actions – CI/CD Pipeline

  A GitHub Actions workflow (.github/workflows/docker-publish.yml) automates the process of:

  1.Building the Docker image

  2.Tagging it properly

  3.Pushing it to the GitHub Container Registry (GHCR)

  Workflow Highlights
  - name: Checkout repository
    uses: actions/checkout@v3

  - name: Log in to GHCR
    uses: docker/login-action@v3
    with:
      registry: ghcr.io
      username: ${{ github.actor }}
      password: ${{ secrets.GITHUB_TOKEN }}

  - name: Build and push Docker image
    uses: docker/build-push-action@v4
    with:
      context: .
      push: true
      tags: ghcr.io/${{ github.repository_owner }}/devops-assessment-next:latest
  
4.Verify GHCR Image
  Once the workflow succeeds, verify your image in

    You can also pull it manually:
    docker pull ghcr.io/adithya160-gif/devops-assessment-next:latest

    Then test locally:
    docker run -p 3000:3000 ghcr.io/adithya160-gif/devops-assessment-next:latest
    
5.Deploy on Minikube (Kubernetes)
  Start Minikube
  minikube start --driver=docker

  Verify Node
  kubectl get nodes
  
k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nextjs-deployment
  labels:
    app: nextjs-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nextjs-app
  template:
    metadata:
      labels:
        app: nextjs-app
    spec:
      containers:
        - name: nextjs-container
          image: ghcr.io/adithya160-gif/devops-assessment-next:latest
          ports:
            - containerPort: 3000
          readinessProbe:
            httpGet:
              path: /
              port: 3000
            initialDelaySeconds: 10
            periodSeconds: 5
          livenessProbe:
            httpGet:
              path: /
              port: 3000
            initialDelaySeconds: 15
            periodSeconds: 10
            
k8s/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nextjs-service
spec:
  selector:
    app: nextjs-app
  ports:
    - protocol: TCP
      port: 3000
      targetP
      port: 3000
  type: NodePort

Apply Kubernetes Manifests
    kubectl apply -f k8s/deployment.yaml
    kubectl apply -f k8s/service.yaml
Check resources:
    kubectl get pods
    kubectl get svc

Access Application
    minikube service nextjs-service
or find the IP:
    minikube ip

