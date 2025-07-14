pipeline {
    agent any

    environment {
        REGISTRY = credentials('docker-registry-url')
        IMAGE_NAME = credentials('docker-image-name')
        IMAGE_TAG = credentials('docker-image-tag')
        APP_PORT = credentials('docker-app-port')
    }

    options {
        skipStagesAfterUnstable()
        timestamps()
    }

    stages {
        stage('Clone Repository') {
            steps {
                git(
                    url: 'https://github.com/MostafaSadab/node-express-hello-devfile-no-dockerfile.git',
                    branch: 'main',
                    credentialsId: 'github-token'
                )
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh '''
                        docker build -t $REGISTRY/$IMAGE_NAME:$IMAGE_TAG .
                    '''
                }
            }
        }

        stage('Push to Local Docker Registry') {
            steps {
                script {
                    sh '''
                        docker push $REGISTRY/$IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Run and Verify Image') {
            steps {
                script {
                    sh '''
                        docker rm -f test-run || true
                        docker pull $REGISTRY/$IMAGE_NAME:$IMAGE_TAG
                        docker run -d --name test-run -p $APP_PORT:$APP_PORT $REGISTRY/$IMAGE_NAME:$IMAGE_TAG
                        sleep 5
                        curl --fail http://localhost:$APP_PORT || (echo 'App failed to respond.' && exit 1)
                    '''
                }
            }
        }
    }

    post {
        always {
            echo "Cleaning up test container"
            sh 'docker rm -f test-run || true'
        }
        success {
            echo "Pipeline completed successfully."
        }
        failure {
            echo "Pipeline failed. Check logs for details."
        }
    }
}
