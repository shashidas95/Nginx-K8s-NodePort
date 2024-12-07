# Task 1

- make a index.html file
- make a docker file for this index file
  - base image will be nginx
  - this index file should be served via nginx server
- make a kubernetes deployment file for that docker image
- expose the deployment via nodeport

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

# FileSyncer Project

**FileSyncer** is a Kubernetes deployment consisting of two containers that create files in a shared volume (`emptyDir`) every minute. The project demonstrates how to use Kubernetes volumes, containers, and cron jobs to create files in a synchronized manner.

## Project Overview

- **Container 1** creates log files (e.g., `log1.txt`, `log2.txt`, etc.) every minute.
- **Container 2** creates data files (e.g., `file1.txt`, `file2.txt`, etc.) every minute.
- Both containers mount the same `emptyDir` volume at `/mount` in their file system.
- Files are created by custom shell commands running as cron jobs in each container.

## Kubernetes Setup

This project uses the following resources:

- **Deployment**: A Kubernetes deployment with 2 containers.
- **Shared Volume**: An `emptyDir` volume mounted by both containers at `/mount`.

## Files

- **deployment.yaml**: The Kubernetes manifest file that defines the deployment and shared volume.
- **container 1**: Creates files named `log1.txt`, `log2.txt`, etc.
- **container 2**: Creates files named `file1.txt`, `file2.txt`, etc.

## Prerequisites

Ensure you have the following installed:

- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) for interacting with your Kubernetes cluster.
- A running Kubernetes cluster (you can use Minikube, KIND, or a cloud-based Kubernetes service).
- Access to your Kubernetes cluster.

## Deployment Instructions

1. **Clone the repository**:

   ```bash
   git clone https://github.com/shashidas95/Nginx-k8s-Nodeport.git
   cd Nginx-k8s-Nodeport/FileSyncer/k8s
   ```

2. **Apply the Kubernetes deployment**:

   ```bash
   kubectl apply -f deployment.yaml
   ```

3. **Check the running pods**:
   After applying the deployment, verify that the pods are running:

   ```bash
   kubectl get pods
   ```

4. **Access the containers**:
   To view the files created by the containers, exec into either container and check the `/mount` directory:
   ```bash
   kubectl exec -it multi-container-deployment-5589df565f-fqkrg -c container-1 -- sh
   ls /mount
   ```

kubectl exec -it multi-container-app-bf4f455df-s6spb -- /bin/sh ls /mount/

/ # ls
bin etc lib mnt opt root sbin sys usr
dev home media mount proc run srv tmp var
/ # cd mount
/mount # ls
file1.log file2.log file3.log log1.log log2.log log3.log
/mount # cat file2.log
2: Sat Dec 7 06:29:10 UTC 2024
/mount #

Similarly, check the files in container 2:

```bash
kubectl exec -it <pod name> -c container-2 -- sh
ls /mount
```

5. **Check logs** (Optional):
   If you want to troubleshoot or see the cron job execution, you can view the logs of the containers:

   ```bash
    kubectl exec -it <pod_name> -c container-1 -- cat /mount/log/log1.txt
    kubectl exec -it <pod_name> -c container-2 -- cat /mount/file/file1.txt

   ```

   ```bash
   kubectl logs <pod-name> -c container-1
   kubectl logs <pod-name> -c container-2
   ```

## File Naming

- **Container 1**: Creates files named `log1.txt`, `log2.txt`, `log3.txt`, and so on.
- **Container 2**: Creates files named `file1.txt`, `file2.txt`, `file3.txt`, and so on.

The cron jobs run every minute to create new files with incrementing numbers.

## Notes

- The `emptyDir` volume is ephemeral, meaning that the data will be lost when the pod is deleted or restarted. If you need persistent storage, consider using a PersistentVolumeClaim (PVC).
- Ensure that the cron jobs are working properly by checking the logs or verifying the creation of files in `/mount`.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

```

### Key Sections Explained:
1. **Project Overview**: Describes the project’s goal and what each container does.
2. **Kubernetes Setup**: Describes the resources used (deployment and shared volume).
3. **Prerequisites**: Lists the tools needed to run the project.
4. **Deployment Instructions**: Provides the commands to clone the repo, apply the deployment, and check the pod status.
5. **File Naming**: Details the file creation process in each container.
6. **License**: This is a placeholder for a license (adjust as necessary).

```
