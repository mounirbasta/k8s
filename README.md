# Kubernetes GitOps Repository with ArgoCD, Monitoring, and CI/CD

## 📋 Overview

This repository implements a complete GitOps-managed Kubernetes environment using ArgoCD for continuous deployment. The infrastructure includes a Kubernetes cluster with comprehensive monitoring, security scanning, and automated deployment pipelines.

> **⚠️ Important Note**: This is currently a development environment running on the main branch. For production use, consider implementing branch protection and environment segregation.

## 🏗️ Architecture Components

### Core Infrastructure
- **Kubernetes Cluster**: Provisioned via Terraform (GCP GKE)
- **Python App**: Python application containing info about users and products
- **NGINX Ingress Controller**: Installed and configured for traffic routing
- **ArgoCD**: GitOps tool for continuous deployment and synchronization
- **Monitoring Stack**: Comprehensive observability suite
- **Cert-Manager**: Automated TLS certificate management

### Monitoring & Observability Stack
- **Grafana with Custom Dashboards**:
  - K8S Dashboard: Kubernetes cluster visualization and management
  - Logs / App Dashboard: Application performance and log analysis
  - Node Exporter Full: Complete node-level metrics and hardware monitoring
- **Alloy**: Metrics and logs collection agent
- **Prometheus**: Metrics storage and alerting
- **Loki**: Log aggregation system
- **Kube-state-metrics**: Kubernetes cluster metrics exporter

## 🔗 Access URLs

| Service | URL | Purpose | SSL Status |
|---------|-----|---------|------------|
| ArgoCD | https://wwwargo.com | GitOps deployment management | **TLS (Let's Encrypt)** |
| Dashboard | http://136.119.151.203/dashboards | Grafana monitoring dashboards | HTTP |
| Python Application | https://py7hon.com/ | Deployed application endpoint | **TLS (Let's Encrypt)** |

## 📁 Repository Structure

```
.
├── README
├── argocd
│   ├── README
│   └── app-dev-application.yaml
├── cert-manager
│   ├── cluster-issuer-prod.yaml
│   ├── cluster-issuer-staging.yaml
│   └── cert-manager-install.yaml
├── configmaps
│   ├── alloy-config.yaml
│   ├── app-config.yaml
│   ├── grafana-datasources.yaml
│   ├── loki-config.yaml
│   └── prometheus-config.yaml
├── daemonset
│   ├── alloy-daemonset.yaml
│   └── app-daemonset.yaml
├── deployments
│   ├── app-deployment.yaml
│   ├── grafana-deployment.yaml
│   ├── kube-state-metrics-deployment.yaml
│   ├── loki-deployment.yaml
│   └── prometheus-deployment.yaml
├── ingresses
│   ├── alloy-dev.yaml
│   ├── app-dev.yaml
│   ├── argo-dev.yaml
│   └── grafana-dev.yaml
├── namespaces
│   ├── app-namespace.yaml
│   ├── cert-manager-namespace.yaml
│   ├── kube-state-metrics-namespace.yaml
│   └── monitoring-namespace.yaml
├── rbac
│   ├── alloy-rbac.yaml
│   └── kube-state-metrics-rbac.yaml
├── secrets
│   ├── app-secrets.yaml
│   └── grafana-secrets.yaml
└── services
    ├── alloy-service.yaml
    ├── app-service.yaml
    ├── grafana-service.yaml
    ├── kube-state-metrics-service.yaml
    ├── loki-service.yaml
    └── prometheus-service.yaml

11 directories, 36 files
```

## 🚀 Getting Started

### Prerequisites

1. **Clone and Setup Terraform Infrastructure**:
   ```bash
   git clone https://github.com/mounirbasta/terraform
   cd terraform
   # Follow Terraform repository instructions to provision the cluster
   ```

2. **Authenticate to GCP Kubernetes Cluster**:
   ```bash
   gcloud auth login
   gcloud auth configure-docker
   gcloud container clusters get-credentials app-cluster \
     --zone us-central1-a \
     --project {your-project-name}
   ```

3. **Clone This Repository**:
   ```bash
   git clone {this-repo-url}
   cd {repository-name}
   ```

### Initial Setup

1. **Apply Namespaces**:
   ```bash
   kubectl apply -f namespaces/
   ```

2. **Install NGINX Ingress Controller**:
   ```bash
   # Add NGINX Ingress Controller
   kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
   ```

3. **Install Cert-Manager**:
   ```bash
   # Install cert-manager CRDs
   kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.0/cert-manager.crds.yaml

   # Add cert-manager Helm repository
   helm repo add jetstack https://charts.jetstack.io
   helm repo update

   # Install cert-manager
   helm install cert-manager jetstack/cert-manager \
     --namespace cert-manager \
     --create-namespace \
     --version v1.13.0

   # Verify installation
   kubectl get pods -n cert-manager
   ```

4. **Configure Let's Encrypt Cluster Issuer**:
   ```bash
   # Apply production cluster issuer
   kubectl apply -f cert-manager/cluster-issuer-prod.yaml
   ```

5. **Deploy kube-state-metrics**:
   ```bash
   # Deploy kube-state-metrics for Kubernetes cluster metrics
   kubectl apply -f https://raw.githubusercontent.com/kubernetes/kube-state-metrics/master/examples/standard/service-account.yaml
   kubectl apply -f https://raw.githubusercontent.com/kubernetes/kube-state-metrics/master/examples/standard/cluster-role.yaml
   kubectl apply -f https://raw.githubusercontent.com/kubernetes/kube-state-metrics/master/examples/standard/cluster-role-binding.yaml
   kubectl apply -f https://raw.githubusercontent.com/kubernetes/kube-state-metrics/master/examples/standard/deployment.yaml
   kubectl apply -f https://raw.githubusercontent.com/kubernetes/kube-state-metrics/master/examples/standard/service.yaml
   ```

6. **Deploy Monitoring Stack**:
   ```bash
   kubectl apply -f configmaps/
   kubectl apply -f secrets/
   kubectl apply -f rbac/
   kubectl apply -f daemonset/
   kubectl apply -f deployments/
   kubectl apply -f services/
   kubectl apply -f ingresses/
   ```

7. **Bootstrap ArgoCD**:
   ```bash
   kubectl create namespace argocd
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   
   # Get ArgoCD admin password
   kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
   ```

8. **Deploy ArgoCD Application**:
   ```bash
   kubectl apply -f argocd/app-dev-application.yaml
   ```

## 🔐 SSL/TLS Configuration

### Cert-Manager Setup

The cluster uses **cert-manager** for automated TLS certificate management with Let's Encrypt:

#### Installation Method Used:
```bash
# This is the method used in the current environment:
kubectl create namespace cert-manager

# Install using the official manifest (method used based on your replicaset names)
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.0/cert-manager.yaml
```

#### Cluster Issuer Configuration:
Create `cert-manager/cluster-issuer-prod.yaml`:
```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: your-email@example.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx
```

### Application Ingress with TLS

The Python application ingress is configured with TLS using cert-manager:

#### app-dev.yaml:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-server-ingress
  namespace: app-dev
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - py7hon.com
    secretName: app-tls-secret
  rules:
  - host: py7hon.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app
            port:
              number: 443
```

> **⚠️ SSL Note**: The application currently shows a fake/self-signed certificate because this is a development environment. In production, ensure:
> - Use a Certificate from a trusted source

### Verifying Certificate Status

```bash
# Check certificate status
kubectl get certificates -A

# Check certificate requests
kubectl get certificaterequests -A

# Check cluster issuers
kubectl get clusterissuer

# Check order status (Let's Encrypt)
kubectl get orders -A
```

## 🎯 Application Information

### Python Application
- **Source Repository**: https://github.com/mounirbasta/task
- **Production URL**: https://py7hon.com/
- **Deployment Method**: GitOps via ArgoCD
- **SSL**: TLS-enabled via Let's Encrypt (development mode with fake cert)

## 🔄 GitOps Workflow

### Repository Structure Philosophy
This is a GitOps repository where:
- Each application should have its own CI pipeline in its source repository
- Application CI pipelines should use GitHub API to update image tags in this repository
- This repository serves as the single source of truth for cluster state
- ArgoCD automatically syncs the cluster with this repository

### Application CI Pipeline Requirements
Each application's CI pipeline should:

1. **Build and Test**: Build container image and run tests
2. **Security Scan**: Perform vulnerability scanning
3. **Update GitOps Repo**: Use GitHub API to update image tags in this repository

```bash
# Example: Update image tag via GitHub API
curl -X PUT \
  -H "Authorization: token $GITHUB_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "message": "Update app image to v1.2.3",
    "content": "'$(base64 -w 0 deployment.yaml)'",
    "sha": "existing_file_sha"
  }' \
  https://api.github.com/repos/owner/repo/contents/applications/app/deployment.yaml
```

## 🔧 Pipeline Integrations

### Security Scanning Pipeline
- **Tool**: Trivy
- **Purpose**: Container vulnerability scanning
- **Trigger**: On every commit and PR
- **Action**: Blocks deployment if critical vulnerabilities detected

### Health Check Pipeline (paste1)
- **Purpose**: Automated deployment health verification
- **Trigger**: Via GitHub webhooks on ArgoCD sync
- **Features**:
  - Health checks against deployed application
  - Service endpoint validation
  - Resource utilization monitoring
  - Automatic rollback on failure (ArgoCD will revert to last working commit)

## 🎣 GitHub Webhook Configuration

### Manual Webhook Setup (via GitHub UI)
1. Navigate to **Repository Settings → Webhooks → Add webhook**
2. **Configuration**:
   - **Payload URL**: `http://wwwargo.com/api/webhook`
   - **Content type**: `application/json`
   - **Secret**: [Your webhook secret]
   - **Events**: Just the push event
   - **Active**: ✓

**Purpose**: Enables push-based ArgoCD synchronization  
**Behavior**: Automatically triggers ArgoCD sync on push to main branch

## 📊 Monitoring Architecture

### Kube-state-metrics
Kube-state-metrics is a service that listens to the Kubernetes API server and generates metrics about the state of various objects:
- **Deployments**: Replica counts, status, and updates
- **Nodes**: Capacity, allocatable resources, and conditions
- **Pods**: Status, resource requests, and limits
- **Services**: Cluster IPs, ports, and selectors
- **PersistentVolumes**: Capacity, access modes, and status

### Alloy Configuration
Alloy is deployed as a DaemonSet to collect:
- **Metrics**: Scraped from Prometheus endpoints including kube-state-metrics
- **Logs**: Collected from all pods and nodes
- **Transport**: Forwarded to Loki and Prometheus

### Grafana Dashboards
Pre-configured dashboards include:
- **K8S Dashboard**: Kubernetes cluster resource usage, pod status, node metrics with kube-state-metrics data
- **Logs / App Dashboard**: Application logs, performance metrics, error rates
- **Node Exporter Full**: CPU, memory, disk, network metrics per node
- **Cluster State Dashboard**: Kubernetes object states and health from kube-state-metrics
- **SSL/TLS Dashboard**: Certificate expiration and SSL status monitoring

### Data Flow
```
Applications → Alloy (DaemonSet) → Prometheus (Metrics) + Loki (Logs) → Grafana (Dashboards)
Kubernetes API → kube-state-metrics → Prometheus → Grafana (Cluster State)
Cert-Manager → Let's Encrypt → TLS Certificates → Secure Ingress
```

## 🔒 Security

### Authentication & Access

```bash
# GCP Authentication
gcloud auth login
gcloud config set project [PROJECT_ID]
gcloud container clusters get-credentials [CLUSTER_NAME] --zone [ZONE]

# ArgoCD Access
kubectl port-forward svc/argocd-server -n argocd 8080:443
# Username: admin
# Password: kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

### TLS & Certificate Security
- **Cert-Manager**: Automated TLS certificate management
- **Let's Encrypt**: Free SSL certificates for development
- **Production Readiness**: Currently using fake certificates for development
- **Certificate Rotation**: Automatic renewal handled by cert-manager

### Vulnerability Management
- **Trivy Scanning**: Integrated in CI pipeline
- **Image Security**: Regular vulnerability scans
- **Access Control**: GCP IAM integrated RBAC

## 🛠️ Application Management

### Adding New Applications with TLS
1. Create Application Manifests in appropriate directories
2. Add cert-manager annotations to ingress for automatic TLS
3. Update ArgoCD Application configuration
4. Configure CI/CD in application repository to update this GitOps repo

### Example TLS-enabled Ingress
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
  namespace: app-dev
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - myapp.example.com
    secretName: myapp-tls-secret
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-app
            port:
              number: 80
```

### Health Monitoring Pipeline
The paste1 health check pipeline:
- Monitors deployment health automatically
- Triggers on webhook events from ArgoCD
- If health checks fail, ArgoCD automatically reverts to the last working commit
- Provides deployment validation and automatic rollback

## 🔧 Troubleshooting

### Common Commands

```bash
# Check ArgoCD status
kubectl get applications -n argocd
argocd app sync [app-name]

# Check NGINX Ingress
kubectl get pods -n ingress-nginx
kubectl get ing -A

# Access Monitoring
kubectl port-forward -n monitoring svc/grafana 3000:3000

# Check Alloy logs
kubectl logs -l app=alloy -n monitoring

# Check kube-state-metrics
kubectl get pods -n kube-system -l app.kubernetes.io/name=kube-state-metrics
kubectl logs -n kube-system -l app.kubernetes.io/name=kube-state-metrics

# Check cert-manager
kubectl get pods -n cert-manager
kubectl get certificates -A
kubectl get certificaterequests -A

# Verify deployments
kubectl get deployments -n app-dev
kubectl get deployments -n monitoring
```

### Getting ArgoCD Password

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

### Cert-Manager Troubleshooting

```bash
# Check cert-manager status
kubectl get pods -n cert-manager

# Check certificate status
kubectl get certificates -A

# Check challenges (ACME)
kubectl get challenges -A

# View certificate events
kubectl describe certificate app-tls-secret -n app-dev

# Check cluster issuer
kubectl describe clusterissuer letsencrypt-prod
```

### Webhook Troubleshooting
- Verify webhook is active in GitHub repository settings
- Check webhook deliveries for errors
- Ensure ArgoCD ingress is properly configured
- Verify webhook secret matches ArgoCD configuration

### Kube-state-metrics Verification
```bash
# Verify kube-state-metrics is running
kubectl get pods -n kube-system | grep kube-state-metrics

# Check metrics endpoint
kubectl port-forward -n kube-system svc/kube-state-metrics 8080:8080
curl http://localhost:8080/metrics
```

### SSL Certificate Issues
```bash
# If certificates aren't issuing:
# 1. Check domain resolution
nslookup py7hon.com

# 2. Check ingress controller logs
kubectl logs -n ingress-nginx -l app.kubernetes.io/component=controller

# 3. Check cert-manager logs
kubectl logs -n cert-manager -l app.kubernetes.io/instance=cert-manager

# 4. Verify cluster issuer
kubectl describe clusterissuer letsencrypt-prod
```

## 📝 Notes

- **GitOps Principle**: This repository is the single source of truth for cluster state
- **Automated Rollbacks**: Health check failures trigger automatic reversion to working commits
- **Manual Webhook**: Webhook configured manually via GitHub UI for ArgoCD push-based sync
- **Monitoring**: Alloy-based collection with pre-configured Grafana dashboards enhanced by kube-state-metrics
- **Security**: Integrated Trivy scanning and GCP authentication
- **Cluster Metrics**: kube-state-metrics provides detailed Kubernetes object state information for comprehensive monitoring
- **TLS/SSL**: Cert-manager with Let's Encrypt for automated certificate management (currently using fake certificates in development)
- **Certificate Note**: The development environment shows fake certificates. For production, ensure proper domain configuration and ACME challenge accessibility
