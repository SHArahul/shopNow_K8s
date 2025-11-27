## Container Orchestration

This project provides production-ready Kubernetes manifests, HELM charts, and Jenkins pipeline configurations for deploying the ShopNow MERN stack application. The solution ensures high availability, scalability, and automated CI/CD workflows.
Key Features

```
Separate deployments for Frontend, Backend, and MongoDB
Horizontal Pod Autoscaling (HPA) for dynamic scaling
ConfigMaps and Secrets for environment configuration
Persistent storage for MongoDB data
HELM chart for simplified deployment and upgrades
Jenkins pipeline for automated build and deployment
Health checks and readiness probes
Resource limits and requests for optimal performance
Network policies for enhanced security

Architecture
┌─────────────────────────────────────────────────────────┐
│                    Kubernetes Cluster                    │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌──────────────┐      ┌──────────────┐               │
│  │   Ingress    │      │   Service    │               │
│  │  Controller  │──────│   (LoadBalancer/NodePort)    │
│  └──────────────┘      └──────────────┘               │
│         │                      │                        │
│         ├──────────────────────┼────────────────┐      │
│         │                      │                 │      │
│  ┌──────▼──────┐       ┌──────▼──────┐   ┌─────▼────┐│
│  │  Frontend   │       │   Backend   │   │ MongoDB  ││
│  │   (React)   │       │  (Node.js)  │   │          ││
│  │  Deployment │◄──────│  Deployment │◄──│   StatefulSet││
│  │  (3 replicas)│      │ (3 replicas)│   │  (3 replicas)││
│  └─────────────┘       └─────────────┘   └──────────┘│
│                                                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │         ConfigMaps & Secrets                     │  │
│  └──────────────────────────────────────────────────┘  │
│                                                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │    Persistent Volumes (MongoDB Data)             │  │
│  └──────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

Prerequisites

Required Tools

Kubernetes Cluster: v1.24+ (minikube, EKS, GKE, AKS, or self-hosted)

kubectl: v1.24+

Helm: v3.10+

Docker: v20.10+

Jenkins: v2.400+ (with required plugins)

Git: v2.30+

```
Project Structure
shopNow/
├── k8s/                                    # Kubernetes manifests
│   ├── namespace.yaml                      # Namespace definition
│   ├── configmap.yaml                      # Application configuration
│   ├── secrets.yaml                        # Sensitive data (base64 encoded)
│   ├── mongodb/
│   │   ├── statefulset.yaml               # MongoDB StatefulSet
│   │   ├── service.yaml                   # MongoDB Service
│   │   ├── pvc.yaml                       # Persistent Volume Claims
│   │   └── mongodb-secret.yaml            # MongoDB credentials
│   ├── backend/
│   │   ├── deployment.yaml                # Backend Deployment
│   │   ├── service.yaml                   # Backend Service
│   │   ├── hpa.yaml                       # Horizontal Pod Autoscaler
│   │   └── configmap.yaml                 # Backend-specific config
│   ├── frontend/
│   │   ├── deployment.yaml                # Frontend Deployment
│   │   ├── service.yaml                   # Frontend Service
│   │   └── hpa.yaml                       # Horizontal Pod Autoscaler
│   └── ingress.yaml                        # Ingress configuration
│
├── helm/                                   # HELM chart
│   └── shopnow/
│       ├── Chart.yaml                      # Chart metadata
│       ├── values.yaml                     # Default values
│       ├── values-dev.yaml                 # Development overrides
│       ├── values-staging.yaml             # Staging overrides
│       ├── values-prod.yaml                # Production overrides
│       └── templates/
│           ├── namespace.yaml
│           ├── configmap.yaml
│           ├── secrets.yaml
│           ├── mongodb-statefulset.yaml
│           ├── mongodb-service.yaml
│           ├── backend-deployment.yaml
│           ├── backend-service.yaml
│           ├── backend-hpa.yaml
│           ├── frontend-deployment.yaml
│           ├── frontend-service.yaml
│           ├── frontend-hpa.yaml
│           ├── ingress.yaml
│           ├── _helpers.tpl               # Template helpers
│           └── NOTES.txt                   # Post-install notes
│
├── jenkins/
│   ├── Jenkinsfile                         # Main pipeline
│   ├── Jenkinsfile.build                   # Build-only pipeline
│   └── Jenkinsfile.deploy                  # Deploy-only pipeline
│
├── docker/
│   ├── frontend/
│   │   └── Dockerfile
│   └── backend/
│       └── Dockerfile
│
├── scripts/
│   ├── setup-cluster.sh                    # Cluster setup script
│   ├── deploy.sh                           # Deployment script
│   ├── rollback.sh                         # Rollback script
│   └── cleanup.sh                          # Cleanup script
│
├── docs/
│   ├── architecture.md
│   ├── deployment-guide.md
│   ├── troubleshooting.md
│   └── api-docs.md
│
└── README.md

```

Quick Start
1. Clone the Repository
   
git clone https://github.com/mohanDevOps-arch/shopNow.git

cd shopNow

3. Set Up Kubernetes Cluster
   
bash# For minikube (local development)
minikube start --cpus=4 --memory=8192 --driver=docker

# Enable required addons
minikube addons enable ingress
minikube addons enable metrics-server

3. Deploy with HELM (Recommended)
bash# Create namespace
kubectl create namespace shopnow

# Install/Upgrade with HELM
```
helm upgrade --install shopnow ./helm/shopnow \
  --namespace shopnow \
  --values ./helm/shopnow/values-dev.yaml \
  --create-namespace
4. Verify Deployment
bash# Check all resources
kubectl get all -n shopnow

# Check pod status
kubectl get pods -n shopnow -w

# Get service URLs
kubectl get ingress -n shopnow
Kubernetes Deployment
Manual Deployment Steps
Step 1: Create Namespace
bashkubectl apply -f k8s/namespace.yaml
Step 2: Create Secrets
bash# Create MongoDB secret
kubectl create secret generic mongodb-secret \
  --from-literal=mongodb-root-password=your-root-password \
  --from-literal=mongodb-password=your-password \
  --namespace=shopnow

# Create Docker registry secret (if using private registry)
kubectl create secret docker-registry regcred \
  --docker-server=your-registry-server \
  --docker-username=your-username \
  --docker-password=your-password \
  --docker-email=your-email \
  --namespace=shopnow
Step 3: Deploy MongoDB
bashkubectl apply -f k8s/mongodb/pvc.yaml
kubectl apply -f k8s/mongodb/statefulset.yaml
kubectl apply -f k8s/mongodb/service.yaml
Step 4: Deploy Backend
bashkubectl apply -f k8s/backend/configmap.yaml
kubectl apply -f k8s/backend/deployment.yaml
kubectl apply -f k8s/backend/service.yaml
kubectl apply -f k8s/backend/hpa.yaml
Step 5: Deploy Frontend
bashkubectl apply -f k8s/frontend/deployment.yaml
kubectl apply -f k8s/frontend/service.yaml
kubectl apply -f k8s/frontend/hpa.yaml
Step 6: Configure Ingress
bashkubectl apply -f k8s/ingress.yaml
Verification Commands
bash# Check deployment status
kubectl rollout status deployment/backend -n shopnow
kubectl rollout status deployment/frontend -n shopnow

# View logs
kubectl logs -f deployment/backend -n shopnow
kubectl logs -f deployment/frontend -n shopnow

# Access application
kubectl port-forward svc/frontend 3000:80 -n shopnow
HELM Chart Deployment
Chart Installation
bash# Development environment
helm install shopnow ./helm/shopnow \
  -f ./helm/shopnow/values-dev.yaml \
  -n shopnow --create-namespace

# Staging environment
helm install shopnow ./helm/shopnow \
  -f ./helm/shopnow/values-staging.yaml \
  -n shopnow-staging --create-namespace

# Production environment
helm install shopnow ./helm/shopnow \
  -f ./helm/shopnow/values-prod.yaml \
  -n shopnow-prod --create-namespace
Chart Upgrade
bashhelm upgrade shopnow ./helm/shopnow \
  -f ./helm/shopnow/values-prod.yaml \
  -n shopnow-prod
Chart Rollback
bash# List releases
helm list -n shopnow-prod

# View history
helm history shopnow -n shopnow-prod

# Rollback to previous version
helm rollback shopnow -n shopnow-prod

# Rollback to specific revision
helm rollback shopnow 3 -n shopnow-prod
Chart Uninstallation
bashhelm uninstall shopnow -n shopnow-prod
Customizing Values
Override default values by creating a custom values file:
bashhelm install shopnow ./helm/shopnow \
  -f ./helm/shopnow/values-prod.yaml \
  -f ./my-custom-values.yaml \
  -n shopnow-prod
Jenkins CI/CD Pipeline
Pipeline Overview
The Jenkins pipeline automates the following stages:

Checkout: Clone source code from GitHub
Build: Build Docker images for frontend and backend
Test: Run unit and integration tests
Push: Push Docker images to container registry
Deploy: Deploy to Kubernetes using HELM
Verify: Run smoke tests and health checks
Notify: Send notifications on success/failure
```

Pipeline Setup
1. Create Jenkins Credentials
   
Navigate to Jenkins → Credentials → System → Global credentials and add:

GitHub Token: github-token (Secret text)

Docker Registry: docker-credentials (Username/Password)

Kubeconfig: kubeconfig (Secret file)

Slack Webhook: slack-webhook (Secret text - optional)

2. Create Jenkins Pipeline Job

New Item → Pipeline
Configure:

GitHub project: https://github.com/mohanDevOps-arch/shopNow

Pipeline Definition: Pipeline script from SCM

SCM: Git

Repository URL: https://github.com/mohanDevOps-arch/shopNow.git

Script Path: jenkins/Jenkinsfile

Branch: */main



3. Configure Webhooks

In GitHub repository settings:

Add webhook: http://your-jenkins-url/github-webhook/

Content type: application/json

Events: Push, Pull Request

```
Pipeline Parameters
groovyparameters {
    choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'prod'], description: 'Deployment environment')
    string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'Docker image tag')
    booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Run tests before deployment')
    booleanParam(name: 'SKIP_BUILD', defaultValue: false, description: 'Skip build stage')
}
Running the Pipeline
bash# Trigger pipeline manually
# Jenkins UI → Build with Parameters

# Trigger via webhook (automatic on git push)
git commit -m "Update application"
git push origin main
Pipeline Stages Explained
Build Stage
groovystage('Build Docker Images') {
    steps {
        script {
            docker.build("shopnow-backend:${IMAGE_TAG}", "./backend")
            docker.build("shopnow-frontend:${IMAGE_TAG}", "./frontend")
        }
    }
}
Deploy Stage
groovystage('Deploy to Kubernetes') {
    steps {
        script {
            sh """
                helm upgrade --install shopnow ./helm/shopnow \
                  -f ./helm/shopnow/values-${ENVIRONMENT}.yaml \
                  --set backend.image.tag=${IMAGE_TAG} \
                  --set frontend.image.tag=${IMAGE_TAG} \
                  --namespace shopnow-${ENVIRONMENT}
            """
        }
    }
}
Configuration
Environment Variables
Backend Configuration
yamlNODE_ENV: production
PORT: 5000
MONGODB_URI: mongodb://mongodb-service:27017/shopnow
JWT_SECRET: <your-jwt-secret>
JWT_EXPIRE: 30d
CORS_ORIGIN: https://shopnow.example.com
Frontend Configuration
yamlREACT_APP_API_URL: https://api.shopnow.example.com
REACT_APP_ENV: production
Resource Limits
yaml# Backend
resources:
  requests:
    memory: "256Mi"
    cpu: "250m"
  limits:
    memory: "512Mi"
    cpu: "500m"

# Frontend
resources:
  requests:
    memory: "128Mi"
    cpu: "100m"
  limits:
    memory: "256Mi"
    cpu: "200m"

# MongoDB
resources:
  requests:
    memory: "512Mi"
    cpu: "500m"
  limits:
    memory: "1Gi"
    cpu: "1000m"
Autoscaling Configuration
yaml# Backend HPA
minReplicas: 2
maxReplicas: 10
targetCPUUtilizationPercentage: 70
targetMemoryUtilizationPercentage: 80

# Frontend HPA
minReplicas: 2
maxReplicas: 5
targetCPUUtilizationPercentage: 75
Monitoring & Logging
Prometheus Metrics
bash# Install Prometheus
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring

# Access Prometheus
kubectl port-forward -n monitoring svc/prometheus-kube-prometheus-prometheus 9090:9090
Grafana Dashboards
bash# Access Grafana
kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80

# Default credentials
Username: admin
Password: prom-operator
Application Logs
bash# Stream backend logs
kubectl logs -f deployment/backend -n shopnow

# Stream frontend logs
kubectl logs -f deployment/frontend -n shopnow

# View MongoDB logs
kubectl logs -f statefulset/mongodb -n shopnow

# Aggregate logs from all pods
kubectl logs -l app=backend -n shopnow --tail=100
ELK Stack Integration
yaml# Add filebeat sidecar to deployments
- name: filebeat
  image: docker.elastic.co/beats/filebeat:8.5.0
  volumeMounts:
  - name: logs
    mountPath: /var/log/app
Troubleshooting
Common Issues
Pods Not Starting
bash# Check pod status
kubectl describe pod <pod-name> -n shopnow

# Common causes:
# - Image pull errors: Check imagePullSecrets
# - Resource constraints: Check node resources
# - Configuration errors: Check ConfigMaps/Secrets
Backend Cannot Connect to MongoDB
bash# Verify MongoDB is running
kubectl get pods -l app=mongodb -n shopnow

# Check MongoDB service
kubectl get svc mongodb-service -n shopnow

# Test connectivity
kubectl exec -it deployment/backend -n shopnow -- nc -zv mongodb-service 27017
Frontend Not Accessible
bash# Check ingress configuration
kubectl get ingress -n shopnow
kubectl describe ingress shopnow-ingress -n shopnow

# Check ingress controller logs
kubectl logs -n ingress-nginx deployment/ingress-nginx-controller
HPA Not Scaling
bash# Check metrics server
kubectl top nodes
kubectl top pods -n shopnow

# Check HPA status
kubectl get hpa -n shopnow
kubectl describe hpa backend-hpa -n shopnow
Debug Commands
bash# Get all resources in namespace
kubectl get all -n shopnow

# Check events
kubectl get events -n shopnow --sort-by='.lastTimestamp'

# Execute commands in pod
kubectl exec -it <pod-name> -n shopnow -- /bin/sh

# Check resource usage
kubectl top pods -n shopnow
kubectl top nodes

# View configuration
kubectl get configmap -n shopnow -o yaml
kubectl get secret -n shopnow
Rollback Procedure
bash# Kubernetes rollback
kubectl rollout undo deployment/backend -n shopnow
kubectl rollout undo deployment/frontend -n shopnow

# HELM rollback
helm rollback shopnow -n shopnow

# Verify rollback
kubectl rollout status deployment/backend -n shopnow

```

Security Considerations |
Best Practices Implemented

Secrets stored in Kubernetes Secrets (base64 encoded)

RBAC policies for service accounts

Network policies to restrict traffic

Non-root containers

Read-only root filesystem where possible

Resource limits to prevent DoS

Image scanning in CI pipeline

TLS/SSL for ingress traffic

Pod Security Standards enforced
