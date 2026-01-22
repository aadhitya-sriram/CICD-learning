pipeline {
    agent any

    environment {
        APP_NAME = "docker-hello"
        K8S_NAMESPACE = "default"
        SERVICE_NAME = "docker-hello-service"
        DEPLOYMENT_NAME = "docker-hello"
        CONTAINER_NAME = "app"
    }

    stages {
        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t ${APP_NAME}:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Load Image into Minikube') {
            steps {
                sh '''
                # Load the image from Jenkins Docker daemon into Minikube
                minikube image load ${APP_NAME}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl apply -f k8s/

                # Update the running deployment to the new image tag
                kubectl set image deployment/${DEPLOYMENT_NAME} \
                  ${CONTAINER_NAME}=${APP_NAME}:${BUILD_NUMBER} \
                  -n ${K8S_NAMESPACE}

                # Wait for rollout
                kubectl rollout status deployment/${DEPLOYMENT_NAME} -n ${K8S_NAMESPACE} --timeout=120s
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                # Get the service URL from minikube and hit /health
                URL=$(minikube service ${SERVICE_NAME} --url)
                echo "Service URL: $URL"
                curl -f "$URL/health"
                '''
            }
        }
    }

    post {
        always {
            sh '''
            # Keep the cluster clean-ish (optional): show state for debugging
            kubectl get pods -n ${K8S_NAMESPACE} || true
            kubectl get svc -n ${K8S_NAMESPACE} || true
            '''
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