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
                            export PATH="/var/jenkins_home/bin:$PATH"
                        '''
                    } else {
                        echo "AWS CLI already installed at: ${awsInstalled}"
                    }

                    // Verify installation
                    sh '/var/jenkins_home/bin/aws --version || aws --version'
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
                        aws configure set aws_access_key_id ${AWS_ACCESS_KEY_ID}
                        aws configure set aws_secret_access_key ${AWS_SECRET_ACCESS_KEY}
                        aws configure set region ${AWS_REGION}
                        aws sts get-caller-identity  # Verify credentials
                        aws s3 ls
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
