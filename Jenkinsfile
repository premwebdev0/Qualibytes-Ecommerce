

pipeline {
    agent any

    environment {
        DOCKER_IMAGE_NAME = 'qazsxedc/qbshop-app'
        DOCKER_MIGRATION_IMAGE_NAME = 'qazsxedc/qbshop-migration'
        DOCKER_IMAGE_TAG = "${BUILD_NUMBER}"

        CONTAINER_NAME = "ecommerce-container"
        PORT = "3000"

        GIT_BRANCH = "dev"
    }

    stages {

        stage('Cleanup Workspace') {
            steps {
                script {
                    cleanWs()
                }
            }
        }

        stage('Clone Repository') {
            steps {
                script {
                    git branch: 'main', url: 'https://github.com/premwebdev0/Qualibytes-Ecommerce.git'
                }
            }
        }

        stage('Cleanup Old Docker Resources') {
            steps {
                script {
                    sh "docker image prune -f"
                    sh "docker container prune -f"
                    sh "docker volume prune -f"
                }
            }
        }

        stage('Build Docker Images') {
            parallel {

                stage('Build App Image') {
                    steps {
                        script {
                            docker_build(
                                imageName: env.DOCKER_IMAGE_NAME,
                                imageTag: env.DOCKER_IMAGE_TAG,
                                dockerfile: 'Dockerfile',
                                context: '.'
                            )
                        }
                    }
                }

                stage('Build Migration Image') {
                    steps {
                        script {
                            docker_build(
                                imageName: env.DOCKER_MIGRATION_IMAGE_NAME,
                                imageTag: env.DOCKER_IMAGE_TAG,
                                dockerfile: 'scripts/Dockerfile.migration',
                                context: '.'
                            )
                        }
                    }
                }
            }
        }

        stage('Run Unit Tests') {
            steps {
                script {
                    run_tests()
                }
            }
        }

        stage('Security Scan (Trivy)') {
            steps {
                script {
                    trivy_scan()
                }
            }
        }

        stage('Push Docker Images') {
            parallel {

                stage('Push App Image') {
                    steps {
                        script {
                            docker_push(
                                imageName: env.DOCKER_IMAGE_NAME,
                                imageTag: env.DOCKER_IMAGE_TAG,
                                credentials: 'docker-hub-credentials'
                            )
                        }
                    }
                }

                stage('Push Migration Image') {
                    steps {
                        script {
                            docker_push(
                                imageName: env.DOCKER_MIGRATION_IMAGE_NAME,
                                imageTag: env.DOCKER_IMAGE_TAG,
                                credentials: 'docker-hub-credentials'
                            )
                        }
                    }
                }
            }
        }

        stage('Deploy Docker Container') {
            steps {
                script {

                    sh """
                    docker rm -f ${CONTAINER_NAME} || true
                    docker pull ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}

                    docker run -d \
                    -p ${PORT}:3000 \
                    --name ${CONTAINER_NAME} \
                    ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}
                    """
                }
            }
        }

        stage('Update Kubernetes Manifests') {
            steps {
                script {
                    update_k8s_manifests(
                        imageTag: env.DOCKER_IMAGE_TAG,
                        manifestsPath: 'kubernetes',
                        gitCredentials: 'github-credentials',
                        gitUserName: 'Jenkins CI',
                        gitUserEmail: 'jenkins@ci.local'
                    )
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment Successful!"
            echo "🌐 App running at: http://<JENKINS-IP>:3000"
        }

        failure {
            echo "❌ Pipeline Failed - Check logs"
        }
    }
}
stage('Build Docker Images') {
    parallel {
        stage('Build App Image') {
            steps {
                sh 'docker build -t qbshop-app:latest .'
            }
        }

        stage('Build Migration Image') {
            steps {
                sh 'docker build -t qbshop-migration:latest .'
            }
        }
    }
}
