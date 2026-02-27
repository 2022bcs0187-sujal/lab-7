pipeline {
    agent any

    environment {
        IMAGE_NAME = 'wine-quality'
        CONTAINER_NAME = 'wine-test-container'
        HOST_PORT = '8001'
        CONTAINER_PORT = '8000'
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
                curl -f http://host.docker.internal:${HOST_PORT}/health
                """
            }
        }

        stage('Valid Test') {
            steps {
                sh """
                RESPONSE=\$(curl -s -X POST http://host.docker.internal:${HOST_PORT}/predict \
                -H "Content-Type: application/json" \
                -d '{
                    "fixed_acidity":7.4,
                    "volatile_acidity":0.7,
                    "citric_acid":0.0,
                    "residual_sugar":1.9,
                    "chlorides":0.076,
                    "free_sulfur_dioxide":11.0,
                    "total_sulfur_dioxide":34.0,
                    "density":0.9978,
                    "pH":3.51,
                    "sulphates":0.56,
                    "alcohol":9.4
                }')

                echo "Valid API Response: \$RESPONSE"

                echo \$RESPONSE | grep wine_quality || exit 1
                """
            }
        }

        stage('Invalid Test') {
            steps {
                sh """
                STATUS=\$(curl -s -o /dev/null -w "%{http_code}" \
                -X POST http://host.docker.internal:${HOST_PORT}/predict \
                -H "Content-Type: application/json" \
                -d '{"fixed_acidity":7.4}')

                echo "Invalid Test Status Code: \$STATUS"

                if [ "\$STATUS" -ne 422 ]; then
                    echo "Invalid input test failed"
                    exit 1
                else
                    echo "Invalid input correctly rejected"
                fi
                """
            }
        }

        stage('Stop Container') {
            steps {
                sh "docker rm -f ${CONTAINER_NAME} || true"
            }
        }
    }

    post {
        success {
            echo "Pipeline PASSED: Model inference validated successfully."
        }
        failure {
            echo "Pipeline FAILED: Inference validation failed."
        }
        always {
            sh "docker rm -f ${CONTAINER_NAME} || true"
        }
    }
}