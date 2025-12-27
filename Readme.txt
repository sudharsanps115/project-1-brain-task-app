FINAL PROJECT – 1
MindTrack

Project Overview:

This project demonstrates an end-to-end CI/CD pipeline using Jenkins, Docker, AWS ECR, and Amazon EKS, with CloudWatch monitoring.
The application is containerized, automatically built, pushed to ECR, and deployed to EKS using Jenkin.

Tech Stack Used :

CI/CD: Jenkins
Containerization: Docker, NGINX
Container Registry: AWS ECR
Orchestration: Amazon EKS (Kubernetes)
Monitoring: Amazon CloudWatch
Cloud: AWS (EC2, IAM

CI/CD Flow:

GitHub → Jenkins → Docker Build → ECR → EKS → CloudWatch

1. Code pushed to GitHub
2. Jenkins pulls code and builds Docker image
3. Image pushed to AWS ECR
4. Jenkins deploys app to EKS using kubectl
5. Logs and metrics monitored via CloudWatch

Jenkins Pipeline (Summary)

1. Verify AWS credentials
2. Build Docker image
3. Tag & push image to ECR
4. Update kubeconfig
5. Deploy application to EKS (Deployment & Service)

Deployment:

1. Kubernetes Deployment with 2 replicas
2. Kubernetes Service (LoadBalancer) exposes the app
3. Container runs on port 3000, exposed externally via port 80

Monitoring:

CloudWatch Dashboard used to monitor:

Jenkins EC2 CPU, memory, disk
EKS cluster & control-plane metrics
ECR repository activity

Ensures visibility into CI/CD and runtime performance
