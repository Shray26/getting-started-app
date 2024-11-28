pipeline {
    agent any

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/appu883/getting-started-app.git'
            }
        }
        stage('Image Build') {
            steps {
                sh 'docker image build -t appu883/simple_app:v$BUILD_ID .'
                sh 'docker image tag appu883/simple_app:v$BUILD_ID appu883/simple_app:latest'
            }
        }
        stage('Container Run') {
            steps {
                sh 'docker run -itd --name todoapp -p 3000:3000 appu883/simple_app:latest'
            }
        }
    }
}
