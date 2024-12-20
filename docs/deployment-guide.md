# Deployment Guide
คู่มือการติดตั้งระบบ

## Infrastructure Setup (การติดตั้งโครงสร้างพื้นฐาน)

### Prerequisites
- Google Cloud Account
- Terraform
- kubectl
- Docker
- GitHub Account
- ArgoCD

### Google Cloud Setup
```bash
# 1. Create Project
gcloud projects create marketplace-microservice

# 2. Enable APIs
gcloud services enable container.googleapis.com
gcloud services enable cloudbuild.googleapis.com
gcloud services enable cloudresourcemanager.googleapis.com

# 3. Create Service Account
gcloud iam service-accounts create terraform-sa
```

### Terraform Infrastructure
```hcl
# main.tf
provider "google" {
  project = "marketplace-microservice"
  region  = "asia-southeast1"
}

# GKE Cluster
resource "google_container_cluster" "primary" {
  name     = "marketplace-cluster"
  location = "asia-southeast1"
  
  initial_node_count = 3
  
  node_config {
    machine_type = "e2-standard-2"
    disk_size_gb = 50
  }
}

# Cloud SQL
resource "google_sql_database_instance" "postgres" {
  name             = "marketplace-postgres"
  database_version = "POSTGRES_13"
  region           = "asia-southeast1"
  
  settings {
    tier = "db-f1-micro"
  }
}

# Redis
resource "google_redis_instance" "cache" {
  name           = "marketplace-redis"
  memory_size_gb = 1
  region         = "asia-southeast1"
}
```

## CI/CD Setup (การติดตั้ง CI/CD)

### GitHub Actions Workflow
```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    
    - name: Build and Test
      run: |
        make test
        make build
    
    - name: Push to GCR
      run: |
        docker tag app gcr.io/$PROJECT_ID/app
        docker push gcr.io/$PROJECT_ID/app

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
    - name: Deploy to ArgoCD
      run: |
        argocd app sync marketplace
```

### ArgoCD Application
```yaml
# argocd/application.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: marketplace
spec:
  project: default
  source:
    repoURL: https://github.com/user/marketplace-microservice
    path: k8s
    targetRevision: HEAD
  destination:
    server: https://kubernetes.default.svc
    namespace: marketplace
```

## Kubernetes Resources (ทรัพยากร Kubernetes)

### Namespace
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: marketplace
```

### Service Deployments
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: user-service
        image: gcr.io/project/user-service
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
```

### Kong API Gateway
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: kong-ingress
  annotations:
    kubernetes.io/ingress.class: kong
spec:
  rules:
  - host: api.marketplace.com
    http:
      paths:
      - path: /users
        pathType: Prefix
        backend:
          service:
            name: user-service
            port:
              number: 80
```

## Monitoring Setup (การติดตั้งระบบติดตาม)

### Prometheus
```yaml
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
metadata:
  name: prometheus
spec:
  serviceAccountName: prometheus
  serviceMonitorSelector:
    matchLabels:
      team: frontend
```

### Grafana
```yaml
apiVersion: integreatly.org/v1alpha1
kind: Grafana
metadata:
  name: grafana
spec:
  deployment:
    spec:
      template:
        spec:
          containers:
            - name: grafana
              image: grafana/grafana:latest
```

### Loki
```yaml
apiVersion: loki.grafana.com/v1
kind: LokiStack
metadata:
  name: loki
spec:
  size: 1x.small
  storage:
    schemas:
    - version: v12
      effectiveDate: "2022-06-01"
```

## Security Setup (การติดตั้งระบบความปลอดภัย)

### Network Policies
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

### Secret Management
```yaml
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: gcp-secrets
spec:
  provider: gcp
  parameters:
    secrets: |
      - resourceName: "projects/project-id/secrets/api-key"
        path: "api-key"
```

## Backup Strategy (กลยุทธ์การสำรองข้อมูล)

### Database Backups
```bash
# Automated backup script
#!/bin/bash
DATE=$(date +%Y%m%d)
BACKUP_DIR="/backups"

# PostgreSQL backup
pg_dump -h hostname -U username -d database > $BACKUP_DIR/postgres_$DATE.sql

# MongoDB backup
mongodump --uri="mongodb://username:password@hostname" --out=$BACKUP_DIR/mongo_$DATE

# Upload to GCS
gsutil cp -r $BACKUP_DIR gs://marketplace-backups/
```

## Scaling Strategy (กลยุทธ์การปรับขนาด)

### Horizontal Pod Autoscaling
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: user-service-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: user-service
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 80
```
