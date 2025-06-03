# Browserless Chrome Kubernetes Deployment

[![Deploy to AWS EKS](https://github.com/your-username/browserless-k8s/actions/workflows/deploy-eks.yml/badge.svg)](https://github.com/your-username/browserless-k8s/actions/workflows/deploy-eks.yml)
[![Helm Chart](https://img.shields.io/badge/Helm-v3.12+-blue.svg)](https://helm.sh/)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-v1.24+-blue.svg)](https://kubernetes.io/)

Production-ready Helm chart and CI/CD pipeline for deploying [browserless/chrome](https://github.com/browserless/chrome) to Kubernetes clusters, including AWS EKS and local development environments.

## 🚀 Features

- **Production-ready Helm chart** with comprehensive configuration options
- **Automated CI/CD pipeline** for AWS EKS deployment via GitHub Actions
- **Auto-scaling support** with Horizontal Pod Autoscaler (HPA)
- **Security-focused** with non-root containers and resource limits
- **Health checks** and monitoring endpoints
- **Local development support** for testing and development
- **Internal-only service** for secure cluster communication

## 📋 Table of Contents

- [Quick Start](#-quick-start)
- [Architecture](#-architecture)
- [Prerequisites](#-prerequisites)
- [Local Development](#-local-development)
- [Production Deployment](#-production-deployment)
- [Configuration](#-configuration)
- [Monitoring](#-monitoring)
- [Troubleshooting](#-troubleshooting)
- [Contributing](#-contributing)

## 🏃 Quick Start

### Local Development

```bash
# Clone the repository
git clone https://github.com/your-username/browserless-k8s.git
cd browserless-k8s

# Deploy to local Kubernetes (Docker Desktop/minikube)
kubectl create namespace browserless
helm install browserless ./deployment/helm-chart \
  --namespace browserless \
  --set replicaCount=1 \
  --set resources.requests.cpu=200m \
  --set resources.requests.memory=512Mi \
  --set autoscaling.enabled=false \
  --set browserless.demoMode=true

# Access the service
kubectl port-forward svc/browserless 3000:3000 -n browserless
```

Visit `http://localhost:3000` to access the browserless interface.

### Production Deployment

1. **Configure GitHub Secrets** (see [Production Setup](#production-setup))
2. **Push to main branch** - automatic deployment to production
3. **Push to develop branch** - automatic deployment to staging

## 🏗️ Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   GitHub        │    │   AWS EKS       │    │   Monitoring    │
│   Actions       │───▶│   Cluster       │───▶│   & Logging     │
│                 │    │                 │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         │              ┌─────────────────┐              │
         │              │  Helm Chart     │              │
         └─────────────▶│  - Deployment   │◀─────────────┘
                        │  - Service      │
                        │  - HPA          │
                        │  - ConfigMap    │
                        └─────────────────┘
```

**Key Components:**
- **Helm Chart**: Kubernetes manifests with configurable values
- **GitHub Actions**: Automated CI/CD pipeline
- **AWS EKS**: Managed Kubernetes service
- **HPA**: Auto-scaling based on CPU/memory usage
- **ClusterIP Service**: Internal-only communication

## 📦 Prerequisites

### For Local Development
- Docker Desktop with Kubernetes enabled OR minikube/kind
- kubectl (v1.24+)
- Helm (v3.12+)

### For Production (AWS EKS)
- AWS EKS cluster
- AWS CLI configured
- GitHub repository with required secrets

## 🔧 Local Development

### Setup Local Environment

1. **Start local Kubernetes cluster**:
   ```bash
   # Docker Desktop: Enable Kubernetes in settings
   # OR use minikube
   minikube start --memory=4096 --cpus=2
   
   # OR use kind
   kind create cluster --name browserless
   ```

2. **Deploy with local values**:
   ```bash
   # Create local values file
   cat > values-local.yaml << EOF
   replicaCount: 1
   resources:
     limits:
       cpu: 500m
       memory: 1Gi
     requests:
       cpu: 200m
       memory: 512Mi
   autoscaling:
     enabled: false
   browserless:
     demoMode: true
     concurrentSessions: 5
   EOF
   
   # Deploy
   helm install browserless ./deployment/helm-chart \
     --namespace browserless \
     --create-namespace \
     --values values-local.yaml
   ```

3. **Access the service**:
   ```bash
   kubectl port-forward svc/browserless 3000:3000 -n browserless
   ```

### Development Workflow

```bash
# Make changes to Helm chart
# Test changes
helm upgrade browserless ./deployment/helm-chart \
  --namespace browserless \
  --values values-local.yaml

# View logs
kubectl logs -f deployment/browserless -n browserless

# Cleanup
helm uninstall browserless -n browserless
```

## 🚀 Production Deployment

### GitHub Repository Setup

1. **Required Secrets**:
   ```
   AWS_ACCESS_KEY_ID          # AWS access key for EKS access
   AWS_SECRET_ACCESS_KEY      # AWS secret access key
   BROWSERLESS_TOKEN          # Optional: Authentication token
   SLACK_WEBHOOK_URL          # Optional: Deployment notifications
   ```

2. **Required Variables**:
   ```
   NAMESPACE=browserless              # Kubernetes namespace
   RELEASE_NAME=browserless    # Helm release name
   REPLICA_COUNT=2                    # Number of replicas
   CPU_REQUEST=500m                   # CPU request
   MEMORY_REQUEST=1Gi                 # Memory request
   CPU_LIMIT=1000m                    # CPU limit
   MEMORY_LIMIT=2Gi                   # Memory limit
   AUTOSCALING_ENABLED=true           # Enable auto-scaling
   MIN_REPLICAS=2                     # Minimum replicas
   MAX_REPLICAS=10                    # Maximum replicas
   ```

3. **Update workflow file**:
   ```yaml
   # Update .github/workflows/deploy-eks.yml
   env:
     AWS_REGION: us-west-2              # Your AWS region
     EKS_CLUSTER_NAME: my-eks-cluster   # Your EKS cluster name
   ```

### Deployment Triggers

- **Automatic**: Push to `main` (production) or `develop` (staging)
- **Manual**: GitHub Actions → Run workflow → Select environment

### Manual Production Deployment

```bash
# Configure AWS CLI
aws configure
aws eks update-kubeconfig --name your-cluster-name --region your-region

# Deploy using Helm
helm upgrade --install browserless ./deployment/helm-chart \
  --namespace browserless \
  --create-namespace \
  --set image.tag=latest \
  --set replicaCount=3 \
  --set resources.requests.cpu=500m \
  --set resources.requests.memory=1Gi \
  --set resources.limits.cpu=1000m \
  --set resources.limits.memory=2Gi \
  --set browserless.token="your-production-token" \
  --set autoscaling.enabled=true
```

## ⚙️ Configuration

### Core Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of replicas | `2` |
| `image.repository` | Container image repository | `browserless/chrome` |
| `image.tag` | Container image tag | `latest` |
| `resources.requests.cpu` | CPU request | `500m` |
| `resources.requests.memory` | Memory request | `1Gi` |
| `resources.limits.cpu` | CPU limit | `1000m` |
| `resources.limits.memory` | Memory limit | `2Gi` |

### Browserless Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `browserless.token` | Authentication token | `""` |
| `browserless.concurrentSessions` | Max concurrent sessions | `10` |
| `browserless.queueLength` | Queue length | `10` |
| `browserless.demoMode` | Enable demo mode | `false` |
| `browserless.prebootChrome` | Preboot Chrome instances | `true` |
| `browserless.keepAlive` | Keep connections alive | `true` |

### Auto-scaling Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `autoscaling.enabled` | Enable HPA | `true` |
| `autoscaling.minReplicas` | Minimum replicas | `2` |
| `autoscaling.maxReplicas` | Maximum replicas | `10` |
| `autoscaling.targetCPUUtilizationPercentage` | CPU threshold | `80` |
| `autoscaling.targetMemoryUtilizationPercentage` | Memory threshold | `80` |

### Example Custom Values

```yaml
# production-values.yaml
replicaCount: 3

resources:
  limits:
    cpu: 2000m
    memory: 4Gi
  requests:
    cpu: 1000m
    memory: 2Gi

browserless:
  token: "your-secure-token"
  concurrentSessions: 20
  queueLength: 50
  demoMode: false
  prebootChrome: true

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 20
  targetCPUUtilizationPercentage: 70

ingress:
  enabled: true
  className: "alb"
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internal
    alb.ingress.kubernetes.io/target-type: ip
  hosts:
    - host: browserless.internal.yourdomain.com
      paths:
        - path: /
          pathType: Prefix
```

## 📊 Monitoring

### Health Check Endpoints

- `GET /pressure` - System pressure and resource usage
- `GET /stats` - Usage statistics and metrics
- `GET /config` - Current configuration
- `GET /metrics` - Prometheus metrics (if enabled)

### Monitoring Commands

```bash
# Check deployment status
kubectl get deployments -n browserless
kubectl get pods -n browserless
kubectl get hpa -n browserless

# View logs
kubectl logs -f deployment/browserless -n browserless

# Port forward for testing
kubectl port-forward svc/browserless 3000:3000 -n browserless

# Test health endpoints
curl http://localhost:3000/pressure
curl http://localhost:3000/stats
```

### Resource Monitoring

```bash
# Check resource usage
kubectl top pods -n browserless
kubectl top nodes

# Describe HPA
kubectl describe hpa browserless -n browserless

# Check events
kubectl get events -n browserless --sort-by='.lastTimestamp'
```

## 🔍 Troubleshooting

### Common Issues

**Pod stuck in Pending:**
```bash
kubectl describe pod <pod-name> -n browserless
kubectl get events -n browserless
# Check: Insufficient resources, node selectors, taints
```

**Pod CrashLoopBackOff:**
```bash
kubectl logs <pod-name> -n browserless --previous
# Check: Resource limits, configuration, image issues
```

**Service not accessible:**
```bash
kubectl get endpoints -n browserless
kubectl describe svc browserless -n browserless
# Check: Service selector, pod labels, port configuration
```

**HPA not scaling:**
```bash
kubectl describe hpa browserless -n browserless
kubectl top pods -n browserless
# Check: Metrics server, resource requests, thresholds
```

### Debug Commands

```bash
# Get detailed pod information
kubectl describe pod -l app.kubernetes.io/name=browserless -n browserless

# Check Helm release status
helm status browserless -n browserless

# Validate Helm chart
helm lint ./deployment/helm-chart

# Dry run deployment
helm install browserless ./deployment/helm-chart --dry-run --debug -n browserless
```

## 📁 Repository Structure

```
.
├── .github/
│   └── workflows/
│       └── deploy-eks.yml          # GitHub Actions workflow
├── helm/
│   └── browserless/
│       ├── Chart.yaml              # Chart metadata
│       ├── values.yaml             # Default values
│       └── templates/
│           ├── _helpers.tpl        # Template helpers
│           ├── deployment.yaml     # Kubernetes deployment
│           ├── service.yaml        # Kubernetes service
│           ├── serviceaccount.yaml # Service account
│           ├── hpa.yaml           # Horizontal Pod Autoscaler
│           └── secret.yaml        # Secrets management
├── docs/                          # Additional documentation
├── examples/                      # Example configurations
└── README.md                      # This file
```

## 🤝 Contributing

1. **Fork the repository**
2. **Create a feature branch**: `git checkout -b feature/amazing-feature`
3. **Make your changes** and test locally
4. **Commit your changes**: `git commit -m 'Add amazing feature'`
5. **Push to the branch**: `git push origin feature/amazing-feature`
6. **Open a Pull Request**

### Development Guidelines

- Test changes locally before submitting PR
- Update documentation for configuration changes
- Follow Kubernetes and Helm best practices
- Include appropriate resource limits and security contexts

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🆘 Support

- **Issues**: [GitHub Issues](https://github.com/your-username/browserless-k8s/issues)
- **Discussions**: [GitHub Discussions](https://github.com/your-username/browserless-k8s/discussions)
- **Browserless Documentation**: [browserless.io/docs](https://docs.browserless.io/)

## 📚 Additional Resources

- [Browserless Documentation](https://docs.browserless.io/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Helm Documentation](https://helm.sh/docs/)
- [AWS EKS Documentation](https://docs.aws.amazon.com/eks/)

---

**⭐ If this project helped you, please consider giving it a star!**