FINAL PROJECT – 1
MindTrack

#Project Overview:

This project demonstrates an end-to-end CI/CD pipeline using Jenkins, Docker, AWS ECR, and Amazon EKS, with CloudWatch monitoring.
The application is containerized, automatically built, pushed to ECR, and deployed to EKS using Jenkin.

#Tech Stack Used :

CI/CD: Jenkins
Containerization: Docker, NGINX
Container Registry: AWS ECR
Orchestration: Amazon EKS (Kubernetes)
Monitoring: Amazon CloudWatch
Cloud: AWS (EC2, IAM

#End-to-End CI/CD Flow

GitHub → Jenkins → Docker Build → ECR → EKS → CloudWatch

Developer pushes code to GitHub
Jenkins pulls the code and triggers the pipeline
Docker image is built using Dockerfile
Image is tagged and pushed to Amazon ECR
Jenkins deploys the application to EKS using kubectl
Application runs on Kubernetes with LoadBalancer access
EKS and infrastructure metrics are monitored using CloudWatch

#Docker Implementation

Dockerfile used in this project

FROM nginx:alpine
RUN rm -rf /usr/share/nginx/html/*
COPY dist/ /usr/share/nginx/html/
EXPOSE 3000
CMD ["nginx" , "-g" , "daemon off;"]

Explanation:
Uses lightweight NGINX Alpine image
Serves static application files from dist/
Application runs internally on port 3000

# Jenkins Pipeline (summary)

Final Jenkinsfile used

pipeline {
    agent any

    environment {
        AWS_REGION = "ap-south-1"
        ECR_REPO   = "045998146012.dkr.ecr.ap-south-1.amazonaws.com/brain-task-app"
        IMAGE_TAG  = "latest"
    }

    stages {

        stage('Verify AWS Credentials') {
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                     credentialsId: 'aws']
                ]) {
                    sh '''
                        aws sts get-caller-identity
                    '''
                }
            }
        }

        stage('Docker Build & Tag') {
            steps {
                sh '''
                    docker build -t brain-tasks-app .
                    docker tag brain-tasks-app:latest $ECR_REPO:$IMAGE_TAG
                '''
            }
        }

        stage('Push Image to ECR') {
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                     credentialsId: 'aws']
                ]) {
                    sh '''
                        aws ecr get-login-password --region $AWS_REGION \
                        | docker login --username AWS --password-stdin $ECR_REPO

                        docker push $ECR_REPO:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                     credentialsId: 'aws']
                ]) {
                    sh '''
                        aws eks update-kubeconfig \
                          --name brain-cluster \
                          --region $AWS_REGION

                        kubectl apply -f Deployment.yaml
                        kubectl apply -f Service.yaml
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "CI/CD Pipeline completed successfully!"
        }
        failure {
            echo "CI/CD Pipeline failed."
        }
    }
}

Pipeline Responsibilities:

1. Secure AWS authentication
2. Docker image build & tagging
3. Push image to ECR
4. Deploy application to EKS automatically

#Kubernetes Deployment

Deployment.yaml:

apiVersion: apps/v1
kind: Deployment
metadata:
  name: brain-task-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: brain-task-app
  template:
    metadata:
      labels:
        app: brain-task-app
    spec:
      imagePullSecrets:
      - name: ecr-secret
      containers:
      - name: brain-task-app
        image: 045998146012.dkr.ecr.ap-south-1.amazonaws.com/brain-task-app:latest
        imagePullPolicy: Always
        ports:
        - containerPort: 3000

Highlights:

1. 2 replicas for high availability
2. Always pulls latest image from ECR
3. Runs container on port 3000


#Kubernetes Service

Service.yaml:

apiVersion: v1
kind: Service
metadata:
  name: brain-task-service
spec:
  type: LoadBalancer
  selector:
    app: brain-task
  ports:
    - port: 80
      targetPort: 3000

Result:

1. Public AWS LoadBalancer created
2. External users access app on port 80
3. Traffic forwarded to container port 3000

Monitoring:

CloudWatch Dashboard used to monitor:

Jenkins EC2 CPU, memory, disk
EKS cluster & control-plane metrics
ECR repository activity

Ensures visibility into CI/CD and runtime performance
