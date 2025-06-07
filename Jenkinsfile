pipeline {
    agent any

    environment {
        AWS_CLI = '/var/jenkins_home/bin/aws'
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

        stage('Verify/Install AWS CLI') {
            steps {
                script {
                    // Check if AWS CLI is already installed
                    def awsInstalled = sh(
                        script: 'command -v aws || echo "not-installed"',
                        returnStdout: true
                    ).trim()

                    if (awsInstalled == "not-installed") {
                        echo "AWS CLI not found. Installing..."
                        sh '''
                            # Download and install AWS CLI v2
                            curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
                            unzip awscliv2.zip
                            ./aws/install -i /var/jenkins_home/aws-cli -b /var/jenkins_home/bin
                            rm -rf awscliv2.zip aws/
                            
                            # Add to PATH for current session
                            // export PATH="/var/jenkins_home/bin:$PATH" ###### I tried this way but the path reverted to the default on each time I do login to the container ######
                        '''
                    } else {
                        echo "AWS CLI already installed at: ${awsInstalled}"
                    }

                    // Verify installation
                    sh '''
                        chmod 777 /var/jenkins_home/bin/aws
                        /var/jenkins_home/bin/aws --version || aws --version
                       '''
                }
            }
        }
        
        stage('Configure AWS CLI') {

            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'AWS_JENKINS_CREDENTIAL',
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                ]]) {
                    sh '''
                        echo "Testing AWS CLI..."
                        ${AWS_CLI} configure set aws_access_key_id ${AWS_ACCESS_KEY_ID}
                        ${AWS_CLI} configure set aws_secret_access_key ${AWS_SECRET_ACCESS_KEY}
                        ${AWS_CLI} configure set region ${AWS_REGION}
                        ${AWS_CLI} sts get-caller-identity  # Verify credentials
                        ${AWS_CLI} s3 ls
                    '''
                }
            }
        }
        stage('Login to AWS ECR') {
            steps {
                script {
                    // Log in to AWS ECR
                    sh "${AWS_CLI} ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
                }
            }
        }

        stage('Tag Docker Image') {
            steps {
                script {
                    // Tag the Docker image for ECR
                    sh "docker tag ${ECR_REPO_NAME}:${IMAGE_TAG} ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                script {
                    // Push the Docker image to ECR
                    sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}:${IMAGE_TAG}"
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
