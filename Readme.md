
# Task 1
- make a index.html file
- make a docker file for this index file
	- base image will be nginx
	- this index file should be served via nginx server
-	make a kubernetes deployment file for that docker image
-	expose the deployment via nodeport

upload manifest files,dockerfile,src code to the github


# Nginx-K8s-NodePort

This project demonstrates deploying an Nginx server in a Kubernetes cluster. The server serves a simple static HTML page and is exposed to external traffic using a NodePort service. Docker Hub credentials are securely managed using Kubernetes secrets defined declaratively in `docker-registry-secret.yaml`.

## Features

- Dockerized Nginx with custom static HTML content.
- Kubernetes Deployment and Service manifest files for deployment and exposure.
- Utilizes Docker Hub to pull the custom image.
- NodePort service to expose the application on a specific port.
- Secure image pull with Docker Hub credentials using Kubernetes secrets.

## Prerequisites

1. A running Kubernetes cluster (minikube, kind, or cloud-based cluster).
2. `kubectl` installed and configured to access your cluster.
3. Docker installed for building the image (optional if pulling directly from Docker Hub).
4. An active Docker Hub account.

## Project Structure

```bash
Nginx-K8s-NodePort/
├── Dockerfile
├── index.html
├── k8s/
│   ├── deployment.yaml
│   ├── docker-registry-secret.yaml
└── README.md
```

## Steps to Run

### 1. Clone the Repository

```bash
git clone https://github.com/shashidas95/Nginx-K8s-NodePort.git
cd Nginx-K8s-NodePort
```

### 2. Build and Push the Docker Image

(Optional: Skip this if using the pre-built image `shashidas/nginx:1.1`)

```bash
docker build -t shashidas/nginx:1.1 .
docker push shashidas/nginx:1.1
```

### 3. Configure Docker Registry Secret Declaratively

Update the `docker-registry-secret.yaml` file with your Docker Hub credentials encoded in base64:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: docker-registry-secret
data:
  .dockerconfigjson: <your-encoded-dockerconfigjson>
type: kubernetes.io/dockerconfigjson
```

To generate the `.dockerconfigjson` value:

```bash
echo -n '{"auths":{"https://index.docker.io/v1/":{"username":"<DOCKER_USERNAME>","password":"<DOCKER_PASSWORD>","email":"<DOCKER_EMAIL>"}}}' | base64 -w 0
```

or use

```bash
docker login
ls ~/.docker/config.json
cat ~/.docker/config.json
base64 -i /Users/shashikantadas/.docker/config.json # for mac use -i only
```

Use the output in docker-registry-secret.yaml file

```bash
ewoJImF1dGhzIjogewoJCSJodHRwczovL2luZGV4LmRvY2tlci5pby92MS8iOiB7fQoJfSwKCSJjcmVkc1N0b3JlIjogIm9zeGtleWNoYWluIiwKCSJjdXJyZW50Q29udGV4dCI6ICJkZXNrdG9wLWxpbnV4Igp9
```

Apply the secret manifest:

```bash
kubectl apply -f k8s/docker-registry-secret.yaml
```

### 4. Deploy to Kubernetes

Navigate to the `k8s/` directory and apply the manifests:

```bash
cd k8s
kubectl apply -f deployment.yaml
```

Ensure the secret is referenced in `deployment.yaml` under `imagePullSecrets`:

```yaml
imagePullSecrets:
  - name: docker-registry-secret
```

### 5. Verify the Deployment

Check if the pods are running:

```bash
kubectl get pods
```

Check if the service is created and accessible:

```bash
kubectl get svc
```

### 6. Access the Application

Find the external NodePort by running:

```bash
kubectl describe svc nginx-service
```

```bash
kubectl get endpoints nginx-service
```

in my case the endpoints are

```bash
192.168.171.104:80,
192.168.171.123:80,
192.168.171.126:80
```

use any of them

```bash
http://192.168.171.104:80
```

or

Access the application using the Node's IP and the NodePort:

```bash
http://<node-ip>:<node-port>

```

### 7. Clean Up

To delete the deployment, service, and secret, run:

```bash
kubectl delete -f deployment.yaml
kubectl delete -f docker-registry-secret.yaml
```

## Notes

- Ensure the image `shashidas/nginx:1.1` is public on Docker Hub or configure the Docker Hub credentials properly in the secret.
- Make sure your cluster nodes are accessible from your machine for NodePort services.

## Troubleshooting

- **Pods in CrashLoopBackOff**: Verify the image URL, logs (`kubectl logs <pod-name>`), and secret configuration.
- **Cannot Access Service**: Ensure the NodePort is open, and the cluster node's IP is reachable.
- **Image Pull Errors**: Check if the secret is correctly applied and referenced in `imagePullSecrets`.

---
