pipeline {
    agent any

    environment {
        IMAGE_NAME = "wine-quality"
        CONTAINER_NAME = "wine-test-container"
        PORT = "8000"
    }

    stages {

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                sh "docker build -t ${IMAGE_NAME}:latest ."
            }
        }

        stage('Run Container') {
            steps {
                echo "Running container..."
                sh """
                docker rm -f ${CONTAINER_NAME} || true
                docker run -d -p ${PORT}:${PORT} --name ${CONTAINER_NAME} ${IMAGE_NAME}:latest
                """
            }
        }

        stage('Wait for Service') {
            steps {
                echo "Waiting for API to be ready..."

                sh """
                for i in {1..10}
                do
                    if curl -f http://localhost:${PORT}/health; then
                        echo "Service is up!"
                        exit 0
                    fi
                    echo "Retrying in 3 seconds..."
                    sleep 3
                done

                echo "Service failed to start"
                exit 1
                """
            }
        }

        stage('Valid Inference Test') {
            steps {
                echo "Running valid inference test..."
                sh """
                curl -X POST http://localhost:${PORT}/predict \
                -H "Content-Type: application/json" \
                -d '{"features": [7.4, 0.7, 0.0, 1.9, 0.076, 11.0, 34.0, 0.9978, 3.51, 0.56, 9.4]}'
                """
            }
        }

        stage('Invalid Inference Test') {
            steps {
                echo "Running invalid inference test..."
                sh """
                curl -X POST http://localhost:${PORT}/predict \
                -H "Content-Type: application/json" \
                -d '{"invalid": "data"}' || true
                """
            }
        }
    }

    post {
        always {
            echo "Cleaning up container..."
            sh "docker rm -f ${CONTAINER_NAME} || true"
        }
    }
}