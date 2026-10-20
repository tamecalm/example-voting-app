# Example Voting App on AWS EKS

This repository contains the containerized multi-tier Voting Application packaged and deployed to Amazon EKS using Terraform, Helm, and GitHub Actions.

## Architecture & Components

The application consists of five microservices communicating across the cluster:
- **vote**: Python Flask frontend where users submit votes.
- **redis**: In-memory message queue collecting incoming votes.
- **worker**: .NET background worker consuming votes from Redis and persisting them to PostgreSQL.
- **db**: PostgreSQL database storing vote records on an internal volume.
- **result**: Node.js web application rendering real-time voting results.

## Key Engineering Decisions & Hardening

1. **Worker Container Migration:** Rebuilt the `worker` Dockerfile from deprecated .NET 7 to .NET 8 LTS to resolve build failures and ensure a long-term supported runtime.
2. **Infrastructure as Code:** Provisioned a VPC and an Amazon EKS cluster (Kubernetes v1.30) with managed node groups (`t3.small` instances) using modular Terraform.
3. **Helm Packaging:** Converted raw Kubernetes manifests into a parameterized Helm chart (`helm/voting-app`) with configurable replica counts, ports, and image tags.
4. **CI/CD & Security:** Implemented a GitHub Actions workflow that runs automated Trivy filesystem vulnerability scans before building and publishing multi-arch images to Docker Hub on every commit to `main`.

## Deployment

### Prerequisites
- AWS CLI configured with administrator permissions
- Terraform >= 1.5.0
- kubectl & Helm v3

### 1. Provision EKS Cluster
```bash
cd terraform
terraform init
terraform apply -auto-approve
aws eks update-kubeconfig --region eu-west-1 --name voting-app-cluster
```

### 2. Deploy Application via Helm
```bash
cd ../helm
helm install voting-app ./voting-app -n default
```

### 3. Verify Services
```bash
kubectl get pods,svc -n default
```
Access the vote UI on port `31000` and results UI on port `31001` via worker node public IPs.
