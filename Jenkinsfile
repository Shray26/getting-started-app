pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "appu883/simple_app" // Docker image name
        DOCKER_CREDENTIALS_ID = "dockerhub-token" // Credentials ID for Docker Hub
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

        stage('Cleanup Existing Container') {
            steps {
                echo "Stopping and removing any existing container..."
                sh """
                docker stop todoapp || true
                docker rm todoapp || true
                """
            }
        }

        stage('Run Container') {
            steps {
                echo "Running the Docker container..."
                sh """
                docker run -itd --name todoapp -p 3000:3000 ${DOCKER_IMAGE}:latest
                """
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully. The container is running, and the image has been pushed to Docker Hub."
        }
        failure {
            echo "Pipeline failed. Check the logs for details."
        }
        always {
            echo "Cleaning up unused Docker images..."
            sh "docker image prune -f" // Removes unused Docker images to free up space
        }
    }
}
