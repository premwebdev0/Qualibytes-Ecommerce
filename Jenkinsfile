pipeline {
    agent any

    environment {
        IMAGE_NAME = "qazsxedc/qualibytes-ecommerce:v1"
        CONTAINER_NAME = "ecommerce-container"
        PORT = "3000"
    }

    stages {

        stage('Pull Docker Image') {
            steps {
                sh 'docker pull "qazsxedc/qualibytes-ecommerce:v1"'
            }
        }

       

        stage('Run Container') {
            steps {
                sh '''
               docker run -d \
-p 3000:3000 \
--name ecommerce-container \
qazsxedc/qualibytes-ecommerce:v1
'''
            }
        }
    }

    post {
        success {
            echo "Application deployed successfully!"
            echo "Access it at: http://<JENKINS-SERVER-IP>:3000"
        }
    }
}
