pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'bushbee/train-schedule'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
        DOCKER_CREDENTIALS_ID = 'docker-cred'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
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
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS_ID) {
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push()
                    }
                }
            }
        }

        stage('DeployToProduction') {
            steps {
                script {
                    sh '''
                    docker ps -q --filter "name=train-schedule" | xargs -r docker stop
                    docker ps -a -q --filter "name=train-schedule" | xargs -r docker rm
                    docker pull bushbee/train-schedule:${BUILD_NUMBER}
                    docker run -d --name train-schedule -p 80:80 bushbee/train-schedule:${BUILD_NUMBER}
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Deployment successful!"
        }
        failure {
            echo "Deployment failed. Check logs for details."
        }
    }
}
