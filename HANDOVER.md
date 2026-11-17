# DevOps Assignment – Project Handover

## 1. Project Overview

This project demonstrates an automated CI/CD and GitOps workflow using GitLab CI/CD, Docker, Kubernetes (k3d), and Argo CD.

The application source code and Kubernetes deployment manifests are maintained in separate Git repositories.

The CI/CD workflow runs on a self-hosted GitLab instance. GitHub repositories are used to publish the project source code and GitOps manifests for evaluation.

## 2. Repositories

### GitHub Repositories (Submission)

**Source Repository:**

https://github.com/shekufeh/devops-app

**GitOps Repository:**

https://github.com/shekufeh/devops-app-gitops

Note: The final project updates are pending synchronization to GitHub at the time of preparing this document.

### Internal GitLab Repositories

**Application Repository:**

`http://gitlab.home.arpa/devops-assignment/devops-app.git`

**GitOps Repository:**

`http://gitlab.home.arpa/devops-assignment/devops-app-gitops.git`

The internal GitLab repositories are used by the CI/CD workflow.

## 3. Architecture

The deployment workflow is:

1. Developer pushes application code to the internal GitLab source repository.
2. GitLab CI/CD triggers the pipeline.
3. The Test stage validates the application.
4. The Build stage builds a Docker image.
5. The Push stage publishes the image to GitLab Container Registry.
6. The pipeline updates the image tag in the internal GitLab GitOps repository.
7. Argo CD detects changes in the GitOps repository.
8. Argo CD synchronizes the desired state with the Kubernetes cluster.

## 4. CI/CD Pipeline

The GitLab CI/CD pipeline contains the following stages:

- **Test:** Runs application validation and tests.
- **Build:** Builds the Docker image.
- **Push:** Pushes the image to GitLab Container Registry and updates the GitOps repository.

The Docker image is tagged using `CI_COMMIT_SHA`.

The pipeline automatically updates the GitOps repository with the newly built image tag.

Kubernetes deployment and reconciliation are managed by Argo CD, not directly by GitLab CI/CD.

## 5. Container Registry

The project uses GitLab Container Registry.

**Registry hostname:**

`gitlab.home.arpa:5050`

**Registry IP address:**

`192.168.130.20`

The registry is hosted in the internal lab environment and is used by the CI/CD pipeline to publish application images.

Kubernetes uses an image pull Secret named `gitlab-registry` to authenticate when pulling private images.

## 6. Kubernetes Deployment

The application is deployed in the `devops-app` namespace.

Kubernetes resources:

- Deployment: `devops-app`
- Service: `devops-app` (ClusterIP)
- Ingress: `devops-app`
- ConfigMap: `devops-app-config`
- Image Pull Secret: `gitlab-registry`

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

## 7. Argo CD Configuration

Argo CD Application: `devops-app`

Source repository: Internal GitLab GitOps repository.

Target revision: `main`

Manifest path: `base`

Destination namespace: `devops-app`

Automated synchronization is enabled:

- `prune: true`
- `selfHeal: true`
- `CreateNamespace=true`

Argo CD continuously monitors the GitOps repository and reconciles the Kubernetes cluster with the declared desired state.

## 8. Validation and Testing

The following tests were performed successfully.

### 8.1 Deployment Health

Both application replicas reached `Running` and `Ready` states.

Argo CD reported:

- Sync status: `Synced`
- Health status: `Healthy`

### 8.2 Self-Healing Test

The Deployment was manually scaled from 2 replicas to 1.

Argo CD detected the live-state drift and automatically restored the desired replica count to 2.

### 8.3 Prune Test

A temporary ConfigMap was added to the Git
