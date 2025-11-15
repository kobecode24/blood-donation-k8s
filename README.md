# Blood Donation Kubernetes Deployment

Production-ready Kubernetes deployment for a healthcare microservices Blood Donation System with multi-environment management using Kustomize.

## 🏥 Project Overview

A complete, enterprise-grade Kubernetes setup for managing blood donation operations across development, staging, and production environments.

**Key Features:**
- 🔐 **Security-First:** Pod Security Standards, non-root containers, read-only filesystems, seccomp
- 🌍 **Multi-Environment:** Dev (1 replica), Staging (2 replicas), Production (3 replicas + auto-scaling)
- 📊 **Advanced Features:** Horizontal Pod Autoscaling, Pod Disruption Budgets, TLS/SSL
- 🏗️ **Infrastructure as Code:** Complete Kustomize configuration for all environments
- 💾 **Persistent Storage:** MySQL database with persistent volumes
- 🔄 **Cross-Namespace Communication:** Microservices spanning multiple namespaces

## 📋 Architecture

### Namespaces
- **blood-donation** (Restricted PSS): Microservices, frontends, ingress
- **blood-donation-data** (Privileged PSS): MySQL database

### Microservices (7 Total)

**Backend APIs (4 services):**
- **User Service** (port 3001): User management
- **Campaign Service** (port 3002): Campaign management
- **Appointment Service** (port 3003): Appointment scheduling
- **Analytics Service** (port 3004): System analytics

**Frontend Applications (2 services):**
- **Donor Frontend** (port 3100): Donor portal
- **Admin Frontend** (port 3200): Admin dashboard

**Database:**
- **MySQL 8.0** (port 3306): Data persistence

### Environment Specifications

| Environment | Replicas | Image Tags | TLS | HPAs | PDBs | Use Case |
|-------------|----------|-----------|-----|------|------|----------|
| **Dev** | 1 | `:dev` | ❌ | ❌ | ❌ | Local development |
| **Staging** | 2 | `:staging` | ✅ | ❌ | ✅ | Pre-production testing |
| **Production** | 3 | `:1.0.0` | ✅ | ✅ | ✅ | Production deployment |

## 📁 Project Structure

```
blood-donation-k8s/
├── k8s/                           # Base Kubernetes manifests
│   ├── namespace/                 # Namespace definitions
│   ├── configmaps/                # ConfigMaps (9 total)
│   ├── secrets/                   # Secrets (2 total)
│   ├── storage/                   # PV/PVC definitions
│   ├── database/                  # MySQL deployment
│   ├── backend-services/          # Microservices (4 deployments + 4 services)
│   ├── frontend-services/         # Frontend apps (2 deployments + 2 services)
│   └── ingress/                   # Ingress resources (3 total)
│
├── k8s-kustomize/                 # Kustomize configuration
│   ├── base/                      # Base kustomization (24 resources)
│   └── overlays/
│       ├── dev/                   # Dev environment overlay
│       ├── staging/               # Staging environment overlay (4 PDBs)
│       └── production/            # Production overlay (4 HPAs + 4 PDBs)
│
├── .gitignore                     # Git ignore rules
└── README.md                      # This file
```

## 🚀 Quick Start

### Prerequisites
- Kubernetes cluster (1.20+)
- kubectl configured
- Kustomize 3.8+

### Deploy Dev Environment

```bash
cd blood-donation-k8s

# Preview the manifests
kubectl kustomize ./k8s-kustomize/overlays/dev --load-restrictor=LoadRestrictionsNone

# Apply the configuration
kubectl kustomize ./k8s-kustomize/overlays/dev --load-restrictor=LoadRestrictionsNone | kubectl apply -f -

# Verify deployment
kubectl get deployments -n blood-donation
kubectl get pods -n blood-donation
```

### Deploy Staging Environment

```bash
# Clean up dev
kubectl kustomize ./k8s-kustomize/overlays/dev --load-restrictor=LoadRestrictionsNone | kubectl delete -f - --ignore-not-found

# Deploy staging
kubectl kustomize ./k8s-kustomize/overlays/staging --load-restrictor=LoadRestrictionsNone | kubectl apply -f -

# Verify
kubectl get deployments -n blood-donation -o custom-columns=NAME:.metadata.name,REPLICAS:.spec.replicas
```

### Deploy Production Environment

```bash
# Clean up staging
kubectl kustomize ./k8s-kustomize/overlays/staging --load-restrictor=LoadRestrictionsNone | kubectl delete -f - --ignore-not-found

# Deploy production
kubectl kustomize ./k8s-kustomize/overlays/production --load-restrictor=LoadRestrictionsNone | kubectl apply -f -

# Verify with HPAs and PDBs
kubectl get deployments -n blood-donation
kubectl get hpa -n blood-donation
kubectl get pdb -n blood-donation
```

## 🔐 Security Features

- **Pod Security Standards:** Restricted for app namespace, Privileged for database
- **Security Context:** Non-root users (uid 1000), read-only filesystems, dropped capabilities
- **Network Policies:** Ingress with TLS enforcement (TLSv1.2+)
- **Secrets Management:** Base64-encoded secrets for credentials
- **HIPAA Compliance:** Annotations for healthcare compliance tracking

## 📊 Resource Management

### Microservices
- **Requests:** 100m CPU, 128Mi memory
- **Limits:** 200m CPU, 256Mi memory

### Database (MySQL)
- **Requests:** 250m CPU, 512Mi memory
- **Limits:** 500m CPU, 1Gi memory

### Production Auto-Scaling
- **Min Replicas:** 3
- **Max Replicas:** 6
- **Target CPU:** 70% utilization

## 🔄 Configuration Management

### ConfigMaps (9 total)
- Service configurations (user, campaign, appointment, analytics, admin)
- Frontend configurations (HTML, NGINX config)

### Secrets (2 total)
- MySQL credentials
- Application secrets (JWT, DB password)

### Cross-Namespace Communication
Services reference MySQL via FQDN:
```
mysql-service.blood-donation-data.svc.cluster.local
```

## 📝 Ingress Configuration

Three ingress resources with path-based routing:

1. **blood-donation-ingress** (API Gateway)
   - `/user/(.*)` → user-service:3001
   - `/campaign/(.*)` → campaign-service:3002
   - `/appointment/(.*)` → appointment-service:3003
   - `/analytics/(.*)` → analytics-service:3004

2. **frontend-ingress** (Donor Portal)
   - `/` → donor-frontend-service:3100

3. **admin-ingress** (Admin Dashboard)
   - `/` → admin-frontend-service:3200

## 🎯 Validation

### Pre-Deployment Checks

```powershell
# Verify base resources (24 expected)
kubectl kustomize .\k8s-kustomize\base --load-restrictor=LoadRestrictionsNone | Select-String "^kind:" | Measure-Object

# Verify dev replicas (1 expected)
kubectl kustomize .\k8s-kustomize\overlays\dev --load-restrictor=LoadRestrictionsNone | Select-String "replicas: 1"

# Verify staging replicas (2 expected)
kubectl kustomize .\k8s-kustomize\overlays\staging --load-restrictor=LoadRestrictionsNone | Select-String "replicas: 2"

# Verify production replicas (3 expected)
kubectl kustomize .\k8s-kustomize\overlays\production --load-restrictor=LoadRestrictionsNone | Select-String "replicas: 3"

# Verify image tags
kubectl kustomize .\k8s-kustomize\overlays\dev --load-restrictor=LoadRestrictionsNone | Select-String "newTag: dev"
```

## 🐛 Troubleshooting

### Issue: "Path restriction" error on Windows
**Solution:** Use `--load-restrictor=LoadRestrictionsNone` flag

### Issue: "Dubious ownership" git error
**Solution:** 
```bash
git config --global --add safe.directory /path/to/blood-donation-k8s
```

### Issue: TLS secret warning
**Solution:** Replace placeholder PEM data with real certificates before production

## 📚 Documentation

- `COMPLETE-DEPLOYMENT-GUIDE.md` - Comprehensive deployment guide
- `blood-donation-k8s-exercise-guide.md` - Implementation exercise guide
- `k8s-kustomize/` - Kustomize configuration files

## 🔗 Related Repositories

- [Activite-Kustomize-Multi-Environnements](https://github.com/kobecode24/Activite-Kustomize-Multi-Environnements) - Kustomize activity guide

## 📦 Commits

Total: **56 atomic commits** covering:
- Foundation & Infrastructure (5 commits)
- Database Layer (7 commits)
- Configuration Management (10 commits)
- Backend Microservices (8 commits)
- Frontend Applications (5 commits)
- Networking & Ingress (3 commits)
- Advanced Features (10 commits)
- Kustomize Configuration (4 commits)
- Git Configuration (1 commit)

## 👤 Author

**kobe** - DevOps Engineer  
Email: kobecode24@outlook.com

## 📄 License

This project is part of the DevOps training program.

## 🤝 Contributing

For improvements or issues, please create a pull request or issue on GitHub.

---

**Last Updated:** November 16, 2025  
**Status:** ✅ Production Ready
