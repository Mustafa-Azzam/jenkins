pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = '126157276875'
        AWS_REGION = 'me-south-1'
        ECR_REPO_NAME = 'indana-client.app'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Mustafa-Azzam/jenkins.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${ECR_REPO_NAME}:${IMAGE_TAG} ."
                }
            }
        }

        stage('Configure AWS CLI') {
            agent {
                docker {
                    image 'public.ecr.aws/aws-cli/aws-cli'  // Directly from Docker Hub (no ECR needed)
                    reuseNode true  // Reuse the same workspace
                    args '--entrypoint=""'  // Override entrypoint to allow shell commands
                }
            }
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'AWS_JENKINS_CREDENTIAL',
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                ]]) {
                    sh '''
                        echo "Testing AWS CLI..."
                        aws configure set aws_access_key_id ${AWS_ACCESS_KEY_ID}
                        aws configure set aws_secret_access_key ${AWS_SECRET_ACCESS_KEY}
                        aws configure set region ${AWS_REGION}
                        aws sts get-caller-identity  # Verify credentials
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
