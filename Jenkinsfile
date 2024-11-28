pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "appu883/simple_app"
        DOCKER_CREDENTIALS_ID = "dockerhub-cred" // Use the token's credentials ID
    }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/appu883/getting-started-app.git'
            }
        }
        
        stage('Image Build') {
            steps {
                sh """
                docker image build -t ${DOCKER_IMAGE}:v$BUILD_ID .
                docker image tag ${DOCKER_IMAGE}:v$BUILD_ID ${DOCKER_IMAGE}:latest
                """
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withDockerRegistry([credentialsId: "${DOCKER_CREDENTIALS_ID}", url: 'https://index.docker.io/v1/']) {
                    sh """
                    docker push ${DOCKER_IMAGE}:v$BUILD_ID
                    docker push ${DOCKER_IMAGE}:latest
                    """
                }
            }
        }

        stage('Cleanup Existing Container') {
            steps {
                sh """
                docker stop todoapp || true
                docker rm todoapp || true
                """
            }
        }

        stage('Run Container') {
            steps {
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
            script {
                sh "docker image prune -f" // Optional: Cleans up unused images
            }
        }
    }
}
