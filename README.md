
# Runbook: Deploying HealthSync Solution

---

## Objective
Deploy the HealthSync solution on a Kubernetes cluster using AWS services, ensuring scalability, security, and high availability. This runbook covers the deployment steps, CI/CD pipeline setup, and the integration of metrics and visualization tools.

---

## Prerequisites
1. AWS account with necessary permissions for:
   - Amazon EKS
   - Amazon Redshift
   - Amazon QuickSight
   - Amazon Cognito
   - Amazon Elastic Load Balancer (ELB)
   - AWS Secrets Manager or GitHub Secrets for storing credentials.
2. Kubernetes CLI tools installed:
   - `kubectl`
   - `eksctl`
3. Docker installed locally for building images.
4. GitHub repository configured with source code.
5. GitHub Actions configured for CI/CD.
6. Access to Docker Hub or AWS Elastic Container Registry (ECR).

---

## Deployment Architecture

**Request and Data Flow:**
1. Users authenticate via **AWS Cognito**.
2. User requests are routed through **AWS ELB** to appropriate microservices.
3. Services communicate internally within the Kubernetes cluster via REST APIs.
4. Aggregated data is stored in **Amazon Redshift**.
5. Metrics and logs are collected via **Prometheus** and visualized using **Grafana** or **AWS QuickSight**.

**Microservices:**
1. **Patient Record Service** - Manages patient data.
2. **Appointment Scheduling Service** - Handles appointment creation and updates.
3. **Notification Service** - Sends notifications for appointments.
4. **Aggregator Service** - Aggregates data for analytics.

---

## Deployment Steps

### Step 1: Create AWS EKS Cluster
Run the following command to create the Kubernetes cluster:
```bash
eksctl create cluster --name cc-cw-eks --region us-east-2 --nodegroup-name standard-workers --node-type t2.micro --nodes 2 --nodes-min 1 --nodes-max 2 --managed
```

### Step 2: Update KubeConfig
Connect to the newly created cluster:
```bash
aws eks update-kubeconfig --region us-east-2 --name cc-cw-eks
```

### Step 3: Create Docker Secret
Add Docker credentials to the cluster:
```bash
kubectl create secret docker-registry my-docker-secret   --docker-server=docker.io   --docker-username=<dockerusername>   --docker-password=<dockerpassword>   --docker-email=<dockeraccountemail>
```

### Step 4: Apply Metrics Server
Install the Kubernetes metrics server:
```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

### Step 5: Deploy Microservices
1. Create and apply deployment manifests for each microservice (e.g., `patient-record-service.yaml`, `appointment-scheduling-service.yaml`, etc.):
   ```bash
   kubectl apply -f patient-record-service.yaml
   kubectl apply -f appointment-scheduling-service.yaml
   kubectl apply -f notification-service.yaml
   kubectl apply -f aggregator-service.yaml
   ```
2. Verify deployments:
   ```bash
   kubectl get pods
   ```

### Step 6: Configure Ingress Controller
1. Create the `ingress-nginx` namespace:
   ```bash
   kubectl create namespace ingress-nginx
   ```
2. Deploy the NGINX ingress controller:
   ```bash
   curl https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml -o nginx-ingress-controller.yaml
   kubectl apply -f nginx-ingress-controller.yaml
   ```
3. Apply ingress resources:
   ```bash
   kubectl apply -f ingress-resource.yaml
   ```
4. Verify ingress deployment:
   ```bash
   kubectl get pods -n ingress-nginx
   kubectl get svc -n ingress-nginx
   ```

### Step 7: Configure Cluster Autoscaler
Enable autoscaling for the cluster:
```bash
kubectl apply -f https://github.com/kubernetes/autoscaler/releases/download/cluster-autoscaler-1.30.3/cluster-autoscaler-aws.yaml
```

### Step 8: Set Up CI/CD Pipeline
1. Configure **GitHub Actions** with a `CICD.yaml` workflow:
   - Retrieve source code from GitHub.
   - Build and test the application.
   - Build Docker images and push them to Docker Hub/AWS ECR.
   - Deploy the images to the Kubernetes cluster.
2. Ensure secrets are managed using GitHub Secrets or AWS Secrets Manager.

---

## Metrics and Visualization
1. **Metrics Collection**:
   - Install Prometheus in the cluster to collect metrics:
     ```bash
     kubectl apply -f prometheus-config.yaml
     ```
2. **Visualization**:
   - For **Grafana**, deploy it in the cluster and connect it to Prometheus.
   - For **AWS QuickSight**, connect it to Amazon Redshift for dashboard creation.

---

## Verification
1. Check service status:
   ```bash
   kubectl get svc
   ```
2. Verify metrics collection in Prometheus/Grafana.
3. Confirm data ingestion in Amazon Redshift and visualization in QuickSight.
4. Test API endpoints using tools like Postman or curl.

---

## Troubleshooting
1. **Pod Failures**:
   - Check logs:
     ```bash
     kubectl logs <pod-name>
     ```
2. **Service Unreachable**:
   - Verify ingress and service configurations.
   - Check the ingress controller:
     ```bash
     kubectl describe svc ingress-nginx-controller -n ingress-nginx
     ```
3. **Metrics Not Collected**:
   - Verify Prometheus configuration.
   - Check the connection between Prometheus and Grafana/AWS QuickSight.

---

## Contacts
- **DevOps Team**: devops@meditrack.com
- **Support Team**: support@meditrack.com
