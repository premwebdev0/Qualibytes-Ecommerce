pipeline {
    agent any

    environment {
        DOCKER_IMAGE_NAME = 'qazsxedc/qbshop-app'
        DOCKER_MIGRATION_IMAGE_NAME = 'qazsxedc/qbshop-migration'
        DOCKER_IMAGE_TAG = "${BUILD_NUMBER}"

        CONTAINER_NAME = "ecommerce-container"
        PORT = "3000"
    }

    stages {

        stage('Cleanup Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/premwebdev0/Qualibytes-Ecommerce.git'
            }
        }

        stage('Cleanup Old Docker Resources') {
            steps {
                sh "docker image prune -f"
                sh "docker container prune -f"
                sh "docker volume prune -f"
            }
        }

        stage('Build Docker Images') {
            parallel {

                stage('Build App Image') {
                    steps {
                        sh """
                        docker build -t ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} .
                        """
                    }
                }

                stage('Build Migration Image') {
                    steps {
                        sh """
                        docker build -t ${DOCKER_MIGRATION_IMAGE_NAME}:${DOCKER_IMAGE_TAG} -f scripts/Dockerfile.migration .
                        """
                    }
                }
            }
        }

        stage('Run Unit Tests') {
            steps {
                sh "echo 'Run tests here (npm test / mvn test / pytest)'"
            }
        }

        stage('Security Scan (Trivy)') {
            steps {
                sh "echo 'Run trivy scan here'"
                # Example (if installed):
                # sh "trivy image ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
            }
        }

        stage('Push Docker Images') {
            steps {
                sh """
                echo "Login required here if DockerHub push needed"
                docker push ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}
                docker push ${DOCKER_MIGRATION_IMAGE_NAME}:${DOCKER_IMAGE_TAG}
                """
            }
        }

        stage('Deploy Docker Container') {
            steps {
                sh """
                docker rm -f ${CONTAINER_NAME} || true

                docker run -d \
                -p ${PORT}:3000 \
                --name ${CONTAINER_NAME} \
                ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}
                """
            }
        }
    }

    post {
        success {
            echo "✅ Deployment Successful!"
            echo "🌐 App running at: http://<server-ip>:3000"
        }

        failure {
            echo "❌ Pipeline Failed - check logs"
        }
    }
}
