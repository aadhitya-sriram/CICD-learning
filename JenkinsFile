pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t docker-hello-ci .'
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                docker run -d -p 5001:5000 --name test-container docker-hello-ci
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                sleep 5
                curl -f http://localhost:5001/health
                '''
            }
        }
    }

    post {
        always {
            sh 'docker rm -f test-container || true'
        }
    }
}
