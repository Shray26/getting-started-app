pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "shray26/myapp"
        DOCKER_CREDENTIALS_ID = "dockerhub-token"
        SSH_CREDENTIALS_ID = "server-ssh-credentials"
        SERVER_2_USER = "ubuntu"
        SERVER_2_IP = "172.31.2.71"
    }

    stages {
        stage('Git Checkout') {
            steps {
                script {
                    echo "Checking out the repository..."
                    git branch: 'main', url: 'https://github.com/Shray26/my-app.git'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building the Docker image..."
                    sh """
                    docker image build -t ${DOCKER_IMAGE}:v${BUILD_ID} . 
                    docker image tag ${DOCKER_IMAGE}:v${BUILD_ID} ${DOCKER_IMAGE}:latest
                    """
                }
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                script {
                    echo "Pushing Docker image to Docker Hub..."
                    withDockerRegistry(credentialsId: "${DOCKER_CREDENTIALS_ID}", url: 'https://index.docker.io/v1/') {
                        sh """
                        docker push ${DOCKER_IMAGE}:v${BUILD_ID}
                        docker push ${DOCKER_IMAGE}:latest
                        """
                    }
                }
            }
        }

        stage('Deploy to Server 2') {
            steps {
                script {
                    echo "Deploying Docker image to Server 2..."

                    sshagent([SSH_CREDENTIALS_ID]) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ${SERVER_2_USER}@${SERVER_2_IP} '
                            docker pull ${DOCKER_IMAGE}:latest && 
                            docker stop todoapp || true &&
                            docker rm todoapp || true &&
                            docker run -d --name todoapp -p 3000:3000 ${DOCKER_IMAGE}:latest
                        '
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline execution completed."
        }
        success {
            echo "Pipeline executed successfully!"
        }
        failure {
            echo "Pipeline failed, please check the logs."
        }
    }
}
