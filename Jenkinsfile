pipeline {
    agent any

    environment {
        IMAGE_NAME = '2022bcs0187sujal/wine-quality:latest'
        CONTAINER_NAME = 'wine-test-container'
        PORT = '8000'
    }

    stages {

        stage('Pull Image') {
            steps {
                sh 'docker pull $IMAGE_NAME'
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                    docker run -d -p $PORT:8000 --name $CONTAINER_NAME $IMAGE_NAME
                '''
            }
        }

        stage('Wait for Service') {
            steps {
                sh '''
                    echo "Waiting for API..."
                    for i in {1..10}
                    do
                        if curl -s http://localhost:$PORT/health > /dev/null; then
                            echo "Service is ready"
                            exit 0
                        fi
                        sleep 3
                    done
                    echo "Service failed to start"
                    exit 1
                '''
            }
        }

        stage('Valid Inference Test') {
            steps {
                sh '''
                    RESPONSE=$(curl -s -X POST http://localhost:$PORT/predict \
                    -H "Content-Type: application/json" \
                    -d @tests/valid_input.json)

                    echo "Response: $RESPONSE"

                    echo $RESPONSE | grep wine_quality || exit 1
                '''
            }
        }

        stage('Invalid Inference Test') {
            steps {
                sh '''
                    STATUS=$(curl -s -o /dev/null -w "%{http_code}" \
                    -X POST http://localhost:$PORT/predict \
                    -H "Content-Type: application/json" \
                    -d @tests/invalid_input.json)

                    echo "Status Code: $STATUS"

                    if [ "$STATUS" -eq 422 ]; then
                        echo "Invalid input correctly rejected"
                    else
                        echo "Invalid test failed"
                        exit 1
                    fi
                '''
            }
        }

        stage('Stop Container') {
            steps {
                sh '''
                    docker stop $CONTAINER_NAME
                    docker rm $CONTAINER_NAME
                '''
            }
        }
    }

    post {
        always {
            sh 'docker rm -f $CONTAINER_NAME || true'
        }
    }
}