pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "appu883/simple_app"
        DOCKER_CREDENTIALS_ID = "dockerhub-token"  // Replace with your Docker Hub credentials
        SSH_USER = "ubuntu"
        SERVER_IP = "172.31.5.201"  // Replace with your Server 2 IP address
    }

    stages {
        stage('Git Checkout') {
            steps {
                echo "Checking out the repository..."
                git branch: 'main', url: 'https://github.com/appu883/getting-started-app.git'
            }
        }
        
        stage('Image Build') {
            steps {
                echo "Building the Docker image..."
                sh """
                docker image build -t ${DOCKER_IMAGE}:v${BUILD_ID} .
                docker image tag ${DOCKER_IMAGE}:v${BUILD_ID} ${DOCKER_IMAGE}:latest
                """
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo "Pushing Docker image to Docker Hub..."
                withDockerRegistry([credentialsId: "${DOCKER_CREDENTIALS_ID}", url: 'https://index.docker.io/v1/']) {
                    sh """
                    docker push ${DOCKER_IMAGE}:v${BUILD_ID}
                    docker push ${DOCKER_IMAGE}:latest
                    """
                }
            }
        }

        stage('Deploy to Remote Server') {
            steps {
                echo "Deploying the app to the remote server..."
                sh """
                ssh ${SSH_USER}@${SERVER_IP} << EOF
                docker pull ${DOCKER_IMAGE}:latest
                docker stop todoapp || true
                docker rm todoapp || true
                docker run -itd --name todoapp -p 3000:3000 ${DOCKER_IMAGE}:latest
                EOF
                """
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully. The app is deployed and accessible via the browser."
        }
        failure {
            echo "Pipeline failed. Check the logs for details."
        }
        always {
            echo "Cleaning up unused Docker images..."
            sh "docker image prune -f"
        }
    }
}
