# DevOps Assignment – Handover

## 1. Project Overview

This project demonstrates a CI/CD and GitOps workflow using GitLab CI, Docker, Kubernetes (k3d), and Argo CD.

The application source code and Kubernetes deployment manifests are maintained in separate Git repositories.

## 2. Repositories

**Application repository**

`http://gitlab.home.arpa/devops-assignment/devops-app.git`

**GitOps repository**

`http://gitlab.home.arpa/devops-assignment/devops-app-gitops.git`

## 3. Architecture

The deployment workflow is:

1. Developer pushes application code to the source repository.
2. GitLab CI runs application tests.
3. GitLab CI builds the Docker image.
4. The image is pushed to the GitLab Container Registry.
5. The pipeline updates the image tag in the GitOps repository.
6. Argo CD detects the GitOps change.
7. Argo CD synchronizes the desired state with the Kubernetes cluster.

## 4. CI/CD Pipeline

The GitLab CI pipeline contains the following stages:

- **Test:** Runs application validation.
- **Build:** Builds the Docker image.
- **Push:** Pushes the image to GitLab Container Registry and updates the GitOps repository.

The Docker image is tagged using `CI_COMMIT_SHA`.

The pipeline updates the GitOps repository automatically. Kubernetes deployment is managed by Argo CD, not directly by GitLab CI.

## 5. Kubernetes Deployment

The application is deployed in the `devops-app` namespace.

Kubernetes resources:

- Deployment: `devops-app`
- Service: `devops-app` (ClusterIP)
- Ingress: `devops-app`
- ConfigMap: `devops-app-config`
- Image pull Secret: `gitlab-registry`

Deployment configuration:

- Replicas: 2
- Readiness probe: HTTP `/`
- Liveness probe: HTTP `/`
- CPU request: 100m
- CPU limit: 500m
- Memory request: 64Mi
- Memory limit: 128Mi

Application environment:

`APP_ENV=staging`

Ingress hostname:

`devops-app.local`

Ingress controller: Traefik.

## 6. Argo CD Configuration

Argo CD Application: `devops-app`

Source repository:

`devops-app-gitops`

Target revision: `main`

Manifest path: `base`

Destination namespace: `devops-app`

Automated synchronization is enabled:

- `prune: true`
- `selfHeal: true`
- `CreateNamespace=true`

## 7. Validation and Testing

The following tests were performed:

### Deployment Health

Both application replicas reached `Running` and `Ready` state.

Argo CD reported:

- Sync status: Synced
- Health status: Healthy

### Self-Healing Test

The Deployment was manually scaled from 2 replicas to 1.

Argo CD detected the live-state drift and restored the desired replica count to 2.

### Prune Test

A temporary ConfigMap was added to the GitOps repository and successfully created by Argo CD.

The ConfigMap was then removed from Git. With pruning enabled, Argo CD automatically deleted the ConfigMap from the Kubernetes cluster.

The Application returned to `Synced` and `Healthy`.

## 8. Useful Commands

Check application pods:

```bash
kubectl get pods -n devops-app -o wide
```

Check Deployment:

```bash
kubectl get deployment -n devops-app
```

Check Service and Ingress:

```bash
kubectl get svc,ingress -n devops-app
```

Check Argo CD Application:

```bash
kubectl get application devops-app -n argocd
```

Check Argo CD synchronization policy:

```bash
kubectl get application devops-app -n argocd \
  -o jsonpath='{.spec.syncPolicy}{"\n"}'
```

## 9. Notes

- GitLab CI is responsible for testing, building, publishing images, and updating GitOps manifests.
- Argo CD is responsible for deploying and reconciling Kubernetes resources.
- Registry credentials are stored in a Kubernetes image pull Secret.
- No registry credentials or access tokens should be committed to Git.


