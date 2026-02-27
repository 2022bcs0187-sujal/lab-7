pipeline {
    agent any

    environment {
        IMAGE_NAME = "wine-quality"
        CONTAINER_NAME = "wine-test-container"
        HOST_PORT = "8001"
        CONTAINER_PORT = "8000"
    }

    stages {

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:latest ."
            }
        }

        stage('Run Container') {
            steps {
                sh """
                docker rm -f ${CONTAINER_NAME} || true
                docker run -d -p ${HOST_PORT}:${CONTAINER_PORT} \
                --name ${CONTAINER_NAME} ${IMAGE_NAME}:latest
                """
            }
        }

        stage('Wait for Service') {
            steps {
                sh """
                echo "Waiting for API..."
                sleep 5
                curl -f http://localhost:${HOST_PORT}/health
                """
            }
        }

        stage('Valid Test') {
            steps {
                sh """
                curl -X POST http://localhost:${HOST_PORT}/predict \
                -H "Content-Type: application/json" \
                -d '{"features":[7.4,0.7,0.0,1.9,0.076,11.0,34.0,0.9978,3.51,0.56,9.4]}'
                """
            }
        }
    }

    post {
        always {
            sh "docker rm -f ${CONTAINER_NAME} || true"
        }
    }
}