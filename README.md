# ai-task-platform-infra

Infrastructure repository for the AI Task Processing Platform (GitOps source of truth for Argo CD).

## Structure
```
k8s/
  namespace.yaml
  mongo-deployment.yaml / mongo-service.yaml
  redis-deployment.yaml / redis-service.yaml
  backend-configmap.yaml / backend-secret.yaml
  backend-deployment.yaml / backend-service.yaml
  worker-deployment.yaml / worker-hpa.yaml
  frontend-deployment.yaml / frontend-service.yaml
  ingress.yaml
argocd/
  application.yaml
```

## How it's used
1. The application repository's CI pipeline builds and pushes `backend`, `worker`,
   and `frontend` images to a container registry, tagged with the commit SHA.
2. CI then updates the `image:` tag in `k8s/backend-deployment.yaml`,
   `k8s/worker-deployment.yaml`, and `k8s/frontend-deployment.yaml` in **this**
   repo and commits/pushes the change.
3. Argo CD (configured with `argocd/application.yaml`, pointed at this repo's
   `k8s/` path) detects the git change and auto-syncs the cluster
   (`automated.prune` + `automated.selfHeal` are enabled).

## Bootstrapping Argo CD
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl apply -f argocd/application.yaml
```

Replace `DOCKERHUB_USERNAME` and `repoURL` placeholders before applying.
