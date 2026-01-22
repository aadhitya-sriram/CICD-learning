pipeline {
    agent any

    environment {
        DOCKERHUB_USER = "aadhitya08" 
        APP_NAME = "docker-hello"
        IMAGE_TAG = "${DOCKERHUB_USER}/${APP_NAME}:${BUILD_NUMBER}"
        KUBECONFIG = "/Users/aadhitya/.kube/config"
    }

    stages {
        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Build & Push Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                        sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                        sh "docker build -t ${IMAGE_TAG} ."
                        sh "docker push ${IMAGE_TAG}"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                # 1. Apply the Deployment/Service YAMLs
                kubectl apply -f k8s/

                # 2. Update the deployment to use the new image from Docker Hub
                kubectl set image deployment/docker-hello \
                  app=${IMAGE_TAG} \
                  -n default

                # 3. Wait for it to finish
                kubectl rollout status deployment/docker-hello -n default --timeout=60s
                '''
            }
        }
    }
}



// pipeline {
//     agent any

//     stages {
//         stage('Checkout') {
//             steps {
//                 checkout scm
//             }
//         }

//         stage('Build Docker Image') {
//             steps {
//                 sh 'docker build -t docker-hello .'
//             }
//         }

//         stage('Run Container') {
//             steps {
//                 sh '''
//                 docker run -d -p 5001:8000 --name test-container docker-hello
//                 '''
//             }
//         }

//         stage('Health Check') {
//             steps {
//                 sh '''
//                 sleep 5
//                 curl -f http://docker:5001/health
//                 '''
//             }
//         }
//     }

//     post {
//         always {
//             sh 'docker rm -f test-container || true'
//         }
//     }
// }