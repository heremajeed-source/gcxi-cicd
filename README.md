# GCXI CI/CD Pipeline (Demo)

Simple CI/CD pipeline demo:

```
GitHub → Jenkins → Docker Build → Docker Hub → Kubernetes → Pod → Service → NGINX Ingress
```

## Files
- `index.html` — demo web page
- `Dockerfile` — builds an nginx image serving index.html
- `k8s/deployment.yaml` — Kubernetes Deployment
- `k8s/service.yaml` — Kubernetes Service (ClusterIP)
- `k8s/ingress.yaml` — NGINX Ingress rule (exposes app at `/demo`)
- `Jenkinsfile` — pipeline definition

## How it works
1. Push code to `main` branch on GitHub.
2. GitHub webhook triggers Jenkins job.
3. Jenkins builds a Docker image tagged with the Jenkins build number.
4. Image is pushed to Docker Hub.
5. Jenkins applies the Kubernetes manifests, updating the Deployment with the new image.
6. Kubernetes rolls out the new Pod; NGINX Ingress serves it at `http://<server-ip>/demo`.

## Setup Requirements (Jenkins)
- Credentials: `dockerhub-creds` (Docker Hub username/password), `github-creds` (GitHub token)
- Jenkins container must have access to `kubectl` and a valid kubeconfig
- A Pipeline job pointing to this repo's `Jenkinsfile`
