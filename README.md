# Wiki.js Kubernetes Deployment with ArgoCD

A production-ready Wiki.js deployment on Kubernetes with GitOps automation using ArgoCD.

## 📋 Overview

This project provides a complete Kubernetes deployment configuration for Wiki.js, a modern and powerful wiki application. The deployment includes security best practices, persistent storage, and is designed to work seamlessly with ArgoCD for GitOps-based continuous deployment.

## ✨ Features

- **Production-ready configuration** with security hardening
- **Persistent storage** with SQLite database
- **Resource management** with requests and limits
- **Health checks** for reliability
- **Security contexts** and non-root execution
- **Network policies** for enhanced security
- **RBAC** configuration
- **Ingress support** for external access
- **ArgoCD compatible** for GitOps workflow

## 🏗️ Architecture

The deployment consists of the following Kubernetes resources:

- **Namespace**: Isolated environment for Wiki.js
- **ConfigMap**: Non-sensitive configuration
- **Secret**: Placeholder for sensitive data
- **PersistentVolume/PVC**: Data persistence
- **Deployment**: Wiki.js application
- **Service**: Internal networking
- **Ingress**: External access (optional)
- **ServiceAccount/RBAC**: Security permissions
- **NetworkPolicy**: Network security

## 📋 Prerequisites

- Kubernetes cluster (v1.19+)
- kubectl configured
- ArgoCD installed and configured
- Ingress controller (if using external access)
- Storage class available (local-storage or equivalent)

## 🚀 Quick Start

### 1. Clone the Repository

```bash
git clone <your-repo-url>
cd wikijs-deployment-argocd
```

### 2. Review Configuration

Edit the following files to match your environment:

- Update `wikijs-ingress` host in the YAML file
- Modify storage paths and sizes if needed
- Adjust resource limits based on your cluster capacity

### 3. Deploy with kubectl

```bash
# Apply all resources
kubectl apply -f wikijs.yaml

# Check deployment status
kubectl get pods -n wikijs
kubectl get pvc -n wikijs
```

### 4. Deploy with ArgoCD

Create an ArgoCD Application:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: wikijs
  namespace: argocd
spec:
  project: default
  source:
    repoURL: <your-git-repo-url>
    path: .
    targetRevision: HEAD
  destination:
    server: https://kubernetes.default.svc
    namespace: wikijs
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
```

## 🔧 Configuration

### Environment Variables

The deployment uses a ConfigMap for configuration:

```yaml
DB_TYPE: "sqlite"
DB_FILEPATH: "/data/wikijs.db"
```

### Storage Configuration

- **Storage Size**: 5Gi (configurable)
- **Access Mode**: ReadWriteOnce
- **Reclaim Policy**: Retain
- **Storage Class**: local-storage

### Resource Limits

```yaml
requests:
  memory: "256Mi"
  cpu: "100m"
limits:
  memory: "512Mi"
  cpu: "500m"
```

### Security Features

- Runs as non-root user (UID 1000)
- Read-only root filesystem where possible
- Dropped capabilities
- Security contexts applied
- Network policies for traffic control

## 🌐 Accessing Wiki.js

### Internal Access (Port Forward)

```bash
kubectl port-forward svc/wikijs-service 8080:80 -n wikijs
```

Then access at: `http://localhost:8080`

### External Access (Ingress)

1. Update the ingress host in the deployment file:
   ```yaml
   host: wiki.yourdomain.com
   ```

2. Configure DNS to point to your ingress controller

3. Access at: `https://wiki.yourdomain.com`

## 🔒 Security Considerations

### Network Policies

The deployment includes a NetworkPolicy that:
- Restricts ingress to ingress controller namespace
- Allows all egress traffic (adjust as needed)

### RBAC

Minimal RBAC configuration:
- ServiceAccount for the application
- Role with limited permissions (configmaps, secrets read-only)
- RoleBinding to associate account with role

### Pod Security

- Non-root execution
- Dropped capabilities
- Resource constraints
- Security contexts

## 🔍 Monitoring and Troubleshooting

### Check Pod Status

```bash
kubectl get pods -n wikijs
kubectl describe pod <pod-name> -n wikijs
```

### View Logs

```bash
kubectl logs -f deployment/wikijs -n wikijs
```

### Check Persistent Volume

```bash
kubectl get pv,pvc -n wikijs
kubectl describe pvc wikijs-pvc -n wikijs
```

### Health Checks

The deployment includes:
- **Liveness Probe**: Checks if the application is running
- **Readiness Probe**: Checks if the application is ready to serve traffic

Both probes use the `/healthz` endpoint on port 3000.

## 🔄 ArgoCD Integration

### Sync Policies

The deployment is designed to work with ArgoCD's automated sync:

- **Auto-prune**: Removes resources not in Git
- **Self-heal**: Automatically fixes drift
- **Create namespace**: Automatically creates the wikijs namespace

### Monitoring Sync Status

```bash
# Check application status
argocd app get wikijs

# Sync manually if needed
argocd app sync wikijs
```

## 📊 Scaling and Performance

### Horizontal Scaling

Currently configured for single replica. To scale:

```yaml
spec:
  replicas: 3  # Increase as needed
```

**Note**: Wiki.js with SQLite doesn't support multiple replicas. Consider switching to PostgreSQL for multi-replica setups.

### Vertical Scaling

Adjust resource requests and limits based on usage:

```yaml
resources:
  requests:
    memory: "512Mi"
    cpu: "200m"
  limits:
    memory: "1Gi"
    cpu: "1000m"
```

## 🗄️ Database Migration

To migrate from SQLite to PostgreSQL:

1. Update ConfigMap with PostgreSQL settings
2. Create PostgreSQL deployment or use external service
3. Add database credentials to Secret
4. Update environment variables in Deployment

## 🛠️ Customization

### Adding Custom Themes

Mount custom themes via ConfigMap or additional PVC:

```yaml
volumeMounts:
- name: custom-themes
  mountPath: /wiki/themes/custom
```

### SSL/TLS Configuration

For HTTPS access, configure cert-manager:

```yaml
annotations:
  cert-manager.io/cluster-issuer: "letsencrypt-prod"
tls:
- hosts:
  - wiki.yourdomain.com
  secretName: wikijs-tls
```

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test the deployment
5. Submit a pull request

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🆘 Support

- **Wiki.js Documentation**: https://docs.requarks.io/
- **Kubernetes Documentation**: https://kubernetes.io/docs/
- **ArgoCD Documentation**: https://argo-cd.readthedocs.io/

## 📈 Changelog

### v1.0.0
- Initial release
- Production-ready Wiki.js deployment
- ArgoCD integration
- Security hardening
- Comprehensive documentation