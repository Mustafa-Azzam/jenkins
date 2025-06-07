pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = '126157276875'
        AWS_REGION       = 'me-south-1'    // Change to your region
        ECR_REPO_NAME = 'indana-client.app'
        IMAGE_TAG = "${env.BUILD_NUMBER}"  // Use Jenkins build number as the image tag
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
                    image 'amazon/aws-cli'
                    reuseNode true  // This will reuse the workspace on the same node
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
                        echo "Configuring AWS CLI..."
                        aws configure set aws_access_key_id ${AWS_ACCESS_KEY_ID}
                        aws configure set aws_secret_access_key ${AWS_SECRET_ACCESS_KEY}
                        aws configure set region ${AWS_REGION}
                        aws sts get-caller-identity  # Test the credentials
                        aws s3 ls
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Docker image successfully built and pushed to ECR with tag ${IMAGE_TAG}!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
