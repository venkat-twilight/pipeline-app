pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'venkattwilight/pipeline-app'
        DOCKER_TAG = "0.1"
        DOCKER_REGISTRY_CREDENTIALS = 'dockerhub-credentials-id'
        SERVER_USER = 'root'
        SERVER_IP = '143.110.251.96'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/venkat-twilight/pipeline-app'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_REGISTRY_CREDENTIALS}") {
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push()
                    }
                }
            }
        }

        stage('Deploy to Digital Ocean') {
            steps {
                sshagent (credentials: ['digital-ocean-ssh-key-id']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${SERVER_USER}@${SERVER_IP} 'docker pull ${DOCKER_IMAGE}:${DOCKER_TAG} && docker stop pipeline-app-container || true && docker rm pipeline-app-container || true && docker run -d --name pipeline-app-container -p 80:3000 ${DOCKER_IMAGE}:${DOCKER_TAG}'
                    """
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
