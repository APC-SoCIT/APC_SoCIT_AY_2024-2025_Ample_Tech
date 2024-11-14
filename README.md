# **Testorch: A Scalable Hybrid Test Orchestration Platform**

## **Table of Contents**

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Software Requirements](#software-requirements)
4. [Hardware Requirements](#hardware-requirements)
5. [Deployment Instructions](#deployment-instructions)
6. [Conclusion](#conclusion)

## **Overview**

Testorch is a scalable platform for orchestrating and managing large-scale performance tests. This guide provides the steps necessary to deploy Testorch, covering both frontend (FE) and backend (BE) services, including test management, load generation, and real-time monitoring.

## **Prerequisites**

Ensure the following are installed and configured on your system:

- **Kubernetes** - A Kubernetes cluster is required to orchestrate and manage Testorch’s containerized services.
- **Helm** - Install Helm for managing Kubernetes applications:
  ```bash
  curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
  ```
- **Docker Desktop or Kubernetes** - Required for running Docker containers and managing local Kubernetes clusters.
- **Caddy** - Caddy server is required for the frontend and backend to wrap the application in a Docker container or package, facilitating secure, production-ready hosting. Install Caddy:
  ```bash
  sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
  curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-archive-keyring.gpg
  curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
  sudo apt update
  sudo apt install caddy
  ```

## **Software Requirements**

- **Frontend** - Next.js, TypeScript, Yarn, Docker.
- **Backend** - NestJS, PostgreSQL, Docker, Drizzle ORM for database migrations.
- **Database** - PostgreSQL, configured for both cloud and local environments.
- **Monitoring Tools** - Grafana and InfluxDB for real-time test monitoring and visualization.

## **Hardware Requirements**

- **CPU** - Minimum 4 cores for the server; recommended 8 cores for larger test loads.
- **Memory** - At least 16 GB of RAM.
- **Storage** - 50 GB of storage for logs and database (expandable based on test scale).
- **Network** - Reliable high-speed network connection for optimal performance, especially with remote load generators.

## **Deployment Instructions**

### 1. Add Helm Repositories

Add the necessary Helm repositories for monitoring and storage setup:

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add perf-setup-helm https://sahanjayawardena9619.github.io/perf-setup/perf-setup-helm
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
helm repo update
```

### 2. Install Namespace and Jenkins on Kubernetes

1. **Install a namespace for Jenkins and other services**:
   ```bash
   helm install namespaces perf-setup-helm/1.0-namespace
   ```

2. **Deploy Jenkins in the `perf-platform` namespace**:
   ```bash
   helm install jenkins perf-setup-helm/2.0-jenkins -n perf-platform
   ```

3. **Verify the Jenkins pods**:
   ```bash
   kubectl get pods -n perf-platform
   ```

4. **Access the Jenkins pod**:
   ```bash
   kubectl exec -it pod/<paste-the-pod-id> -n perf-platform -- /bin/bash
   ```

5. **Retrieve the Jenkins admin password**:
   ```bash
   cat /var/jenkins_home/secrets/initialAdminPassword
   ```

6. **Log into Jenkins at [http://localhost:8080](http://localhost:8080) with the username `admin` and the retrieved password**.

7. **Configure Jenkins**:
   - Install recommended plugins.
   - Navigate to **Build Executor Status** > **Configure Number of executors as 0** (This is to prevent using built-in executors; instead, use Kubernetes Plugin to spawn on-demand clusters).
   
8. **Install the Kubernetes Plugin**:
   - Navigate to **Build Executor Status** > **Clouds** > **Install Kubernetes Plugin** > Restart Jenkins.

9. **Configure Kubernetes in Jenkins**:
   - **Kubernetes URL** - Retrieve by executing `kubectl cluster-info`.
   - **Namespace** - Set to `perf-platform` for on-demand pod creation.
   - **Credentials** - Add your Kubernetes config as a secret file (e.g., `~/.kube/config`):
     - **Add** > **Jenkins** > **Secret file** and select the file from `.kube` folder.
   - **Jenkins URL** - Use `http://jenkins-master.perf-platform:8080`.
   - **Jenkins Tunnel** - Set to `jenkins-master.perf-platform:50000`.

### 3. Install Monitoring Tools

1. **Install Prometheus**:
   ```bash
   helm install prometheus prometheus-community/prometheus -f perf-setup-helm/3.0-prometheus/values.yaml -n monitoring
   ```

2. **Install Loki for log management**:
   ```bash
   helm install loki grafana/loki-stack -f perf-setup-helm/4.0-loki/values.yaml -n monitoring
   ```

3. **Install InfluxDB**:
   ```bash
   helm install influxdb perf-setup-helm/5.0-influx -n monitoring
   ```

4. **Install Grafana**:
   ```bash
   helm install grafana perf-setup-helm/6.0-grafana -n monitoring
   ```

### 4. Install Helm and Role Bindings

1. **Install Helm components in the `perf-platform` namespace**:
   ```bash
   helm install helm setup/9.0-helm -n perf-platform
   ```

2. **Install Role Bindings in the `perf-platform` namespace**:
   ```bash
   helm install role-binding setup/10.0-role-binding -n perf-platform
   ```

### 5. Access Grafana and Import Dashboards

1. Access Grafana at [http://localhost:3000](http://localhost:3000).
2. Import dashboards from the `/grafana-dashboards` directory to visualize performance metrics.

### 6. Deploy Testorch Application

1. **Clone the Testorch repositories**:
   ```bash
   git clone https://github.com/EmsiJoseph/testorch.git
   git clone https://github.com/EmsiJoseph/testorch-be.git
   ```

2. **Frontend (Next.js)**:
   - Run the frontend with:
     ```bash
     cd testorch
     npm install
     npm run dev
     ```

3. **Open a new terminal and start Caddy for the frontend**:
   ```bash
   caddy run
   ```

4. **Backend (NestJS)**:
   - Run the backend with:
     ```bash
     cd ..
     cd testorch-be
     npm install
     npm run start:dev
     ```

5. **Open another terminal and start Caddy for the backend**:
   ```bash
   caddy run
   ```

## **Conclusion**

Following these steps will set up the Testorch platform, deploying both frontend and backend components. After deployment, verify that the frontend interacts with the backend as expected and confirm monitoring functionality in Grafana. For further guidance, refer to the user manuals and official Testorch product documentation.
```
