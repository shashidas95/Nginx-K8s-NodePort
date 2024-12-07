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
