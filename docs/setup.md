# Local Development Setup Guide

This guide provides step-by-step instructions to set up a complete local development environment for the DAGIO ERP project. The primary instructions are for a standard Ubuntu installation.

---

### Note for Windows (WSL2) Users

If you are developing on Ubuntu via WSL2, your setup has a few key differences. Please read this section before proceeding.

1.  **Use Docker Desktop:** You must install [Docker Desktop for Windows](https://www.docker.com/products/docker-desktop/) and enable WSL Integration for your Ubuntu distribution in **Settings > Resources > WSL Integration**. Docker Desktop will manage the Docker engine, not Ubuntu.
2.  **Install Docker CLI Only:** In Step 1.2 below, you will only install the Docker command-line interface (`docker-ce-cli`), not the full engine.
3.  **No `systemctl`:** You will skip Step 1.4, as `systemctl` is not used to manage the Docker service in WSL.

---

## Step 1: Install Core Dependencies

Open your Ubuntu terminal and run the following commands to install the necessary tooling.

1.  **Update your package list:**
    ```bash
    sudo apt-get update && sudo apt-get upgrade -y
    ```

2.  **Install Docker and prerequisites:**
    ```bash
    sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common

    # Add Docker's official GPG key
    sudo install -m 0755 -d /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    sudo chmod a+r /etc/apt/keyrings/docker.gpg

    # Set up the repository
    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
      $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
      sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update

    # For a standard Ubuntu installation:
    sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    ```
    > **Note for WSL users:** If you are using Docker Desktop, you only need the command-line tools. Run this command instead of the one above:
    > `sudo apt-get install -y docker-ce-cli`

3.  **Add your user to the `docker` group:**
    This allows you to run `docker` commands without `sudo`.
    ```bash
    sudo usermod -aG docker ${USER}
    ```
    > **IMPORTANT**: You must **close and reopen your Ubuntu terminal** (or log out and back in) for this change to take effect.
 
4.  **(Standard Ubuntu Only) Verify Docker Service:**
    After logging back in, check that the Docker service is running. WSL users should skip this step.
    ```bash
    sudo systemctl status docker
    ```
    You should see `Active: active (running)`. If not, start it with `sudo systemctl start docker`.

5.  **Install `kubectl`, `k3d`, and `helm`:**
    ```bash
    # Install kubectl (for Kubernetes)
    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
    sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
    rm kubectl

    # Install k3d (for local Kubernetes clusters)
    curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash

    # Install helm (for Kubernetes package management)
    curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
    ```

## Step 2: Clone the Repository

Clone the project to your local machine. All subsequent commands should be run from the project's root directory.

```bash
git clone https://github.com/romusking/dagio.git
cd dagio
```

## Step 3: (Optional) Configure Environment Variables

Some services may require environment variables for configuration (e.g., database credentials, ports, etc.).

- Copy the example environment file if present:
  ```bash
  cp .env.example .env
  ```
- Edit `.env` as needed for your local setup.

> If no `.env.example` is present, refer to the service documentation for required variables.

## Step 4: Launch the Local Infrastructure

1.  **Create the Kubernetes Cluster:**
    This command creates a `k3d` cluster named `erp-dev` and maps port `8081` on your local machine to the cluster's load balancer.
    ```bash
    k3d cluster create erp-dev --port '8081:80@loadbalancer'
    ```

2.  **Deploy Core Services:**
    ```bash
    # Add the Bitnami Helm chart repository
    helm repo add bitnami https://charts.bitnami.com/bitnami
    helm repo update

    # Deploy PostgreSQL
    helm install postgres bitnami/postgresql --set auth.postgresPassword=supersecretpassword

    # Deploy TigerBeetle
    kubectl apply -f infra/tigerbeetle/manifest.yaml
    ```

## Step 5: Verify the Setup

Check that all the pods for our core infrastructure are running correctly. It may take a minute or two for them all to show `Running` or `Completed`.

```bash
kubectl get pods
```

You should see output similar to this:

```
NAME                             READY   STATUS      RESTARTS   AGE
postgres-postgresql-0            1/1     Running     0          60s
tigerbeetle-0                    1/1     Running     0          30s
```

Your local environment is now ready!

## Step 6: Access the Application

Once all pods are running, you can access the ERP application via your browser:

- Open: [http://localhost:8081](http://localhost:8081)
- This should route traffic to the load balancer of your local k3d cluster.

If you have deployed a sample frontend or service, you should see its landing page. If not, you can verify service endpoints using:

```bash
kubectl get svc
```

---

## Step 7: Troubleshooting

If you encounter issues, try the following:

- **Check pod logs:**
  ```bash
  kubectl logs <pod-name>
  ```
- **Check cluster status:**
  ```bash
  k3d cluster list
  kubectl get nodes
  kubectl get events
  ```
- **Restart a pod:**
  ```bash
  kubectl delete pod <pod-name>
  ```
- **Delete and recreate the cluster:**
  ```bash
  k3d cluster delete erp-dev
  k3d cluster create erp-dev --port '8081:80@loadbalancer'
  ```

---

## Step 8: Stopping and Cleaning Up

To stop your local development environment:

- **Stop the cluster (pause resources):**
  ```bash
  k3d cluster stop erp-dev
  ```
- **Start it again later:**
  ```bash
  k3d cluster start erp-dev
  ```
- **Delete the cluster (remove all resources):**
  ```bash
  k3d cluster delete erp-dev
  ```

---

## Additional Resources

- [K3d Documentation](https://k3d.io/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Helm Documentation](https://helm.sh/docs/)
- [Bitnami PostgreSQL Helm Chart](https://artifacthub.io/packages/helm/bitnami/postgresql)

---

Congratulations! Your local DAGIO ERP development environment is now fully set up. You are ready to start building and contributing.

If you have questions or run into issues, please check the project documentation or open an issue on GitHub.