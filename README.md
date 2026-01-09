# Kubernetes Manifests Repository

This repository contains Kubernetes manifests for GitOps deployments via ArgoCD.

## Directory Structure

```
k8s-manifests/
├── base/                    # Base manifests (shared across environments)
│   └── namespace.yaml       # Common namespace definitions
├── environments/
│   ├── development/         # Development environment
│   │   └── <app-name>/
│   │       ├── deployment.yaml
│   │       ├── service.yaml
│   │       └── kustomization.yaml
│   └── production/          # Production environment
│       └── <app-name>/
│           ├── deployment.yaml
│           ├── service.yaml
│           └── kustomization.yaml
└── README.md
```

## How It Works

1. **CI Pipeline** builds and pushes Docker image to GitLab registry
2. **CI Pipeline** updates `deployment.yaml` with new image tag
3. **ArgoCD** detects changes and syncs to Kubernetes cluster

## Adding a New Application

1. Create directory: `environments/<env>/<app-name>/`
2. Add Kubernetes manifests (deployment, service, etc.)
3. Create ArgoCD Application pointing to the directory
4. CI pipeline will update the image tag on each build

## Example Application Structure

```yaml
# environments/development/my-app/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  namespace: daitics-apps-dev
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app
          image: gitlab:5050/applications/my-app:latest
          ports:
            - containerPort: 8080
```

## Environment Namespaces

| Environment | Namespace | ArgoCD Project |
|-------------|-----------|----------------|
| Development | daitics-apps-dev | daitics-core-applications |
| Production | daitics-apps-prod | daitics-core-applications |

## CI/CD Integration

The CI pipeline updates manifests using:
```bash
sed -i "s|image:.*|image: ${NEW_IMAGE}|g" deployment.yaml
git commit -m "Deploy ${APP}:${VERSION} to ${ENV}"
git push
```

ArgoCD automatically syncs changes within 3 minutes (or on manual sync).