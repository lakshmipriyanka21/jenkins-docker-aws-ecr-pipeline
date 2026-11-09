
pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPOSITORY = 'petclinic'
    }

    stages {
        stage('Docker Build') {
            steps {
                bat 'docker build -t petclinic:latest .'
            }
        }

        stage('ECR Login and Push') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-credentials'
                ]]) {
                    bat '''
                    @echo off
                    for /f %%A in ('aws sts get-caller-identity --query Account --output text') do set "AWS_ACCOUNT_ID=%%A"
                    set "ECR_REGISTRY=%AWS_ACCOUNT_ID%.dkr.ecr.%AWS_REGION%.amazonaws.com"

                    aws ecr get-login-password --region %AWS_REGION% | docker login --username AWS --password-stdin %ECR_REGISTRY%
                    if errorlevel 1 exit /b 1

                    docker tag petclinic:latest %ECR_REGISTRY%/%ECR_REPOSITORY%:latest
                    if errorlevel 1 exit /b 1

                    docker push %ECR_REGISTRY%/%ECR_REPOSITORY%:latest
                    if errorlevel 1 exit /b 1
                    '''
                }
            }
        }
    }
}
