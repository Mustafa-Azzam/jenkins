pipeline {
    agent {
        docker {
            image 'docker:dind'           // Uses Docker-in-Docker image
            args '-v /var/run/docker.sock:/var/run/docker.sock'  // Shares host Docker socket
            reuseNode true                // Runs on the same Jenkins worker
        }
    }

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
                    // Build the Docker image with the build number as the tag
                    // def dockerImage = docker.build("${ECR_REPO_NAME}:${IMAGE_TAG}")
                    sh "docker build -t ${ECR_REPO_NAME}:${IMAGE_TAG} ."
                }
            }
        }

        // stage('Configure AWS CLI') {
        //     steps {
        //         // Map AWS credentials to environment variables
        //         withCredentials([[
        //             $class: 'AmazonWebServicesCredentialsBinding',
        //             credentialsId: 'AWS_JENKINS_CREDENTIAL', // Replace with your Jenkins AWS credentials ID
        //             accessKeyVariable: 'AWS_ACCESS_KEY_ID',
        //             secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
        //         ]]) {
        //             sh '''
        //                 echo "Configuring AWS CLI..."
        //                 aws configure set aws_access_key_id ${AWS_ACCESS_KEY_ID}
        //                 aws configure set aws_secret_access_key ${AWS_SECRET_ACCESS_KEY}
        //                 aws configure set region ${AWS_REGION}
        //             '''
        //         }
        //     }
        // }

        // stage('Login to AWS ECR') {
        //     steps {
        //         script {
        //             // Log in to AWS ECR
        //             sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        //         }
        //     }
        // }

        // stage('Tag Docker Image') {
        //     steps {
        //         script {
        //             // Tag the Docker image for ECR
        //             sh "docker tag ${ECR_REPO_NAME}:${IMAGE_TAG} ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}:${IMAGE_TAG}"
        //             // dockerImage.tag("${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}","${IMAGE_TAG}")
        //         }
        //     }
        // }

        // stage('Push Docker Image to ECR') {
        //     steps {
        //         script {
        //             // Push the Docker image to ECR
        //             sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}:${IMAGE_TAG}"
        //         }
        //     }
        // }
        
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
