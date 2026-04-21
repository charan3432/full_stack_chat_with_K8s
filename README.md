# Full Stack Chat Application with Kubernetes

A production-ready Kubernetes deployment configuration for a real-time chat application featuring a React frontend, Node.js backend, and MongoDB database.

## 📋 Table of Contents

- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Deployment Guide](#deployment-guide)
- [Configuration](#configuration)
- [Architecture Details](#architecture-details)
- [Monitoring & Maintenance](#monitoring--maintenance)
- [Troubleshooting](#troubleshooting)
- [Security Considerations](#security-considerations)
- [Contributing](#contributing)

## 🏗️ Architecture

This application follows a microservices architecture with three main components:

```
┌─────────────────┐
│     Ingress     │  ← External Traffic Entry Point
└────────┬────────┘
         │
    ┌────┴────┐
    │         │
┌───▼────┐ ┌──▼──────┐
│Frontend│ │ Backend │
│Service │ │ Service │
└───┬────┘ └──┬──────┘
    │         │
┌───▼────┐ ┌──▼──────┐
│Frontend│ │ Backend │
│  Pod   │ │   Pod   │
└────────┘ └──┬──────┘
              │
         ┌────▼────────┐
         │   MongoDB   │
         │   Service   │
         └────┬────────┘
              │
         ┌────▼────────┐
         │   MongoDB   │
         │     Pod     │
         └────┬────────┘
              │
         ┌────▼────────┐
         │ Persistent  │
         │   Volume    │
         └─────────────┘
```

### Components:

- **Frontend**: Web-based user interface for the chat application
- **Backend**: RESTful API server handling business logic and WebSocket connections
- **MongoDB**: NoSQL database for storing messages, users, and chat history
- **Ingress**: Routes external HTTP/HTTPS traffic to appropriate services
- **Persistent Storage**: Ensures MongoDB data persists across pod restarts

## 📦 Prerequisites

Before deploying, ensure you have the following installed:

- **Kubernetes Cluster** (v1.20+)
  - Local: [Minikube](https://minikube.sigs.k8s.io/), [Kind](https://kind.sigs.k8s.io/), or [Docker Desktop](https://www.docker.com/products/docker-desktop)
  - Cloud: GKE, EKS, AKS, or any managed Kubernetes service

- **kubectl** (v1.20+)
  ```bash
  # Verify installation
  kubectl version --client
  ```

- **Ingress Controller** (NGINX recommended)
  ```bash
  # For Minikube
  minikube addons enable ingress
  
  # For other clusters
  kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/cloud/deploy.yaml
  ```

- **Docker Images** (build or use pre-built)
  - Frontend image
  - Backend image

## 🚀 Quick Start

### 1. Clone the Repository

```bash
git clone https://github.com/charan3432/full_stack_chat_with_K8s.git
cd full_stack_chat_with_K8s
```

### 2. Update Secrets

Edit `secrets.yml` and update the base64-encoded values:

```bash
# Encode your MongoDB connection string
echo -n "mongodb://mongo-service:27017/chatapp" | base64

# Encode your JWT secret
echo -n "your-super-secret-jwt-key" | base64
```

### 3. Update Ingress Configuration

Edit `ingress.yml` and replace `your-domain.com` with your actual domain or use `localhost` for local development.

### 4. Deploy to Kubernetes

```bash
# Apply all configurations in order
kubectl apply -f namespace.yml
kubectl apply -f secrets.yml
kubectl apply -f mongodb-pv.yml
kubectl apply -f mongodb-pvc.yml
kubectl apply -f mongo-deployment.yml
kubectl apply -f mongodb-service.yml
kubectl apply -f backend-deployment.yml
kubectl apply -f backend-service.yml
kubectl apply -f frontend-deployment.yml
kubectl apply -f frontend-service.yml
kubectl apply -f ingress.yml
```

### 5. Verify Deployment

```bash
# Check all pods are running
kubectl get pods -n chat-app

# Check services
kubectl get svc -n chat-app

# Check ingress
kubectl get ingress -n chat-app
```

### 6. Access the Application

For **Minikube**:
```bash
minikube service frontend-service -n chat-app
```

For **Cloud/Production**:
```bash
# Get ingress IP
kubectl get ingress -n chat-app
```

Visit the displayed URL in your browser.

## 📝 Deployment Guide

### Detailed Step-by-Step Deployment

#### Step 1: Create Namespace

```bash
kubectl apply -f namespace.yml
```

This creates an isolated namespace `chat-app` for all resources.

#### Step 2: Configure Secrets

Secrets store sensitive information like database credentials and JWT tokens.

```bash
# View current secrets (base64 encoded)
kubectl get secret chat-secrets -n chat-app -o yaml

# Update secrets
kubectl apply -f secrets.yml
```

**Important**: Never commit real secrets to version control!

#### Step 3: Set Up Persistent Storage

```bash
# Create Persistent Volume
kubectl apply -f mongodb-pv.yml

# Create Persistent Volume Claim
kubectl apply -f mongodb-pvc.yml

# Verify
kubectl get pv
kubectl get pvc -n chat-app
```

#### Step 4: Deploy MongoDB

```bash
# Deploy MongoDB
kubectl apply -f mongo-deployment.yml

# Expose MongoDB service
kubectl apply -f mongodb-service.yml

# Verify MongoDB is running
kubectl get pods -n chat-app -l app=mongodb
kubectl logs -n chat-app -l app=mongodb
```

#### Step 5: Deploy Backend

```bash
# Deploy backend API
kubectl apply -f backend-deployment.yml

# Expose backend service
kubectl apply -f backend-service.yml

# Verify backend is running
kubectl get pods -n chat-app -l app=backend
kubectl logs -n chat-app -l app=backend
```

#### Step 6: Deploy Frontend

```bash
# Deploy frontend
kubectl apply -f frontend-deployment.yml

# Expose frontend service
kubectl apply -f frontend-service.yml

# Verify frontend is running
kubectl get pods -n chat-app -l app=frontend
```

#### Step 7: Configure Ingress

```bash
# Apply ingress rules
kubectl apply -f ingress.yml

# Get ingress details
kubectl describe ingress chat-ingress -n chat-app
```

## ⚙️ Configuration

### Environment Variables

Update the following in your deployment files:

**Backend (`backend-deployment.yml`)**:
```yaml
env:
  - name: MONGODB_URI
    valueFrom:
      secretKeyRef:
        name: chat-secrets
        key: mongodb-uri
  - name: JWT_SECRET
    valueFrom:
      secretKeyRef:
        name: chat-secrets
        key: jwt-secret
  - name: PORT
    value: "5001"
```

**Frontend (`frontend-deployment.yml`)**:
```yaml
env:
  - name: REACT_APP_API_URL
    value: "http://backend-service:5001"
```

### Resource Limits (Recommended)

Add resource limits to prevent pod overconsumption:

```yaml
resources:
  requests:
    memory: "256Mi"
    cpu: "250m"
  limits:
    memory: "512Mi"
    cpu: "500m"
```

### Scaling

Scale your deployments based on load:

```bash
# Scale backend
kubectl scale deployment backend-deployment -n chat-app --replicas=3

# Scale frontend
kubectl scale deployment frontend-deployment -n chat-app --replicas=2

# Auto-scaling (HPA)
kubectl autoscale deployment backend-deployment -n chat-app --cpu-percent=50 --min=2 --max=10
```

## 🏛️ Architecture Details

### Networking

- **Frontend Service**: `ClusterIP` (accessed via Ingress)
- **Backend Service**: `ClusterIP` (accessed via Ingress and Frontend)
- **MongoDB Service**: `ClusterIP` (internal access only)

### Data Persistence

MongoDB uses a PersistentVolume to ensure data survives pod restarts:

- **StorageClass**: Uses default or specify custom
- **Access Mode**: `ReadWriteOnce`
- **Capacity**: 5Gi (configurable in `mongodb-pv.yml`)

### Security

1. **Secrets Management**: Sensitive data stored in Kubernetes Secrets
2. **Network Policies**: Implement network policies to restrict pod-to-pod communication
3. **RBAC**: Configure Role-Based Access Control for service accounts
4. **TLS/SSL**: Configure TLS certificates in Ingress for HTTPS

## 📊 Monitoring & Maintenance

### View Logs

```bash
# Frontend logs
kubectl logs -f -n chat-app -l app=frontend

# Backend logs
kubectl logs -f -n chat-app -l app=backend

# MongoDB logs
kubectl logs -f -n chat-app -l app=mongodb

# All pods in namespace
kubectl logs -f -n chat-app --all-containers=true
```

### Check Pod Status

```bash
# Get pod status
kubectl get pods -n chat-app -o wide

# Describe a specific pod
kubectl describe pod <pod-name> -n chat-app

# Get events
kubectl get events -n chat-app --sort-by='.lastTimestamp'
```

### Execute Commands in Pods

```bash
# Access MongoDB shell
kubectl exec -it -n chat-app <mongodb-pod-name> -- mongosh

# Access backend container shell
kubectl exec -it -n chat-app <backend-pod-name> -- /bin/sh
```

### Health Checks

Add liveness and readiness probes to your deployments:

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 5001
  initialDelaySeconds: 30
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /ready
    port: 5001
  initialDelaySeconds: 5
  periodSeconds: 5
```

## 🔧 Troubleshooting

### Common Issues

#### Pods Not Starting

```bash
# Check pod status
kubectl describe pod <pod-name> -n chat-app

# Common issues:
# - Image pull errors: Verify image names and registry access
# - Insufficient resources: Check cluster capacity
# - CrashLoopBackOff: Check logs for application errors
```

#### Service Not Accessible

```bash
# Verify service endpoints
kubectl get endpoints -n chat-app

# Test service connectivity from within cluster
kubectl run -it --rm debug --image=busybox --restart=Never -n chat-app -- sh
wget -O- http://backend-service:5001/health
```

#### Ingress Not Working

```bash
# Check ingress controller is running
kubectl get pods -n ingress-nginx

# Verify ingress resource
kubectl describe ingress chat-ingress -n chat-app

# For Minikube, ensure ingress addon is enabled
minikube addons list
```

#### MongoDB Connection Issues

```bash
# Verify MongoDB is running
kubectl get pods -n chat-app -l app=mongodb

# Check MongoDB logs
kubectl logs -n chat-app -l app=mongodb

# Test connection from backend pod
kubectl exec -it -n chat-app <backend-pod> -- nc -zv mongo-service 27017
```

### Reset Deployment

```bash
# Delete all resources
kubectl delete namespace chat-app

# Redeploy
kubectl apply -f .
```

## 🔒 Security Considerations

### Best Practices

1. **Use Strong Secrets**: Generate cryptographically secure random strings for JWT secrets
   ```bash
   openssl rand -base64 32
   ```

2. **Enable TLS**: Configure SSL/TLS certificates for production
   ```yaml
   # In ingress.yml
   tls:
     - hosts:
         - chat.yourdomain.com
       secretName: chat-tls-secret
   ```

3. **Network Policies**: Restrict pod-to-pod communication
   ```bash
   kubectl apply -f network-policy.yml
   ```

4. **Update Images Regularly**: Keep base images and dependencies updated

5. **Scan for Vulnerabilities**: Use tools like Trivy or Snyk
   ```bash
   trivy image your-backend-image:latest
   ```

6. **Use Non-Root Containers**: Configure security contexts
   ```yaml
   securityContext:
     runAsNonRoot: true
     runAsUser: 1000
   ```

## 🤝 Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 📧 Contact

For questions or support, please open an issue in the repository.

## 🙏 Acknowledgments

- Kubernetes community for excellent documentation
- Contributors and maintainers of the chat application components

---

**Happy Chatting! 💬**
