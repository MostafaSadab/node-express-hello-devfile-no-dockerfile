pipeline {
    agent any

    environment {
        // Use Jenkins credentials for secure configuration
        REGISTRY = credentials('docker-registry-url')         // e.g., localhost:5000
        IMAGE_NAME = credentials('docker-image-name')         // e.g., hello-node-app
        IMAGE_TAG = credentials('docker-image-tag')           // e.g., latest
        APP_PORT = credentials('docker-app-port')             // e.g., 3000
    }

    options {
        skipStagesAfterUnstable()
        timestamps()
    }

    stages {
        stage('Clone Repository') {
            steps {
                git url: 'https://github.com/MostafaSadab/node-express-hello-devfile-no-dockerfile.git', branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building image ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
                    dockerImage = docker.build("${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }

        stage('Push to Local Docker Registry') {
            steps {
                script {
                    echo "Pushing image to ${REGISTRY}"
                    docker.withRegistry("http://${REGISTRY}") {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Run and Verify Image') {
            steps {
                script {
                    echo "Running container from ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
                    
                    sh """
                        docker rm -f test-run || true
                        docker pull ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                        docker run -d --name test-run -p ${APP_PORT}:${APP_PORT} ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                        sleep 5
                        curl --fail http://localhost:${APP_PORT} || (echo 'App failed to respond.' && exit 1)
                    """
                }
            }
        }
    }

    post {
        always {
            echo "Cleaning up test container"
            sh "docker rm -f test-run || true"
        }
        success {
            echo "Pipeline completed successfully."
        }
        failure {
            echo "Pipeline failed. Check logs for details."
        }
    }
}
