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
                        echo "Verifying AWS credentials..."
                        aws sts get-caller-identity
                    '''
                }
            }
        }

        stage('Docker Build & Tag') {
            steps {
                sh '''
                    echo "Building Docker image..."
                    docker build -t brain-tasks-app .

                    echo "Tagging image for ECR..."
                    docker tag brain-tasks-app:latest $ECR_REPO:$IMAGE_TAG
                '''
            }
        }

        stage('Push Image to ECR') {
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                     credentialsId: 'aws-eks-creds']
                ]) {
                    sh '''
                        echo "Logging in to ECR..."
                        aws ecr get-login-password --region $AWS_REGION \
                          | docker login --username AWS --password-stdin $ECR_REPO

                        echo "Pushing image to ECR..."
                        docker push $ECR_REPO:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                     credentialsId: 'aws-eks-creds']
                ]) {
                    sh '''
                        echo "Updating kubeconfig..."
                        aws eks update-kubeconfig \
                          --name brain-cluster \
                          --region $AWS_REGION

                        echo "Deploying application to EKS..."
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
            echo "CI/CD Pipeline failed. Check logs above."
        }
    }
}
