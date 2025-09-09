pipeline {
agent any


environment {
    DOCKER_IMAGE = 'your-dockerhub-username/your-app-name'
    DOCKER_TAG = "${env.BUILD_NUMBER}"
    DOCKER_CREDENTIALS_ID = 'docker-cred' // Jenkins credentials ID
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
                docker.withRegistry('<https://index.docker.io/v1/>', DOCKER_CREDENTIALS_ID) {
                    docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push()
                }
            }
        }
    }

    stage('DeployToProduction') {
        steps {
            script {
                // Stop and remove old container
                sh '''
                docker ps -q --filter "name=your-app-name" | xargs -r docker stop
                docker ps -a -q --filter "name=your-app-name" | xargs -r docker rm
                '''

                // Pull and run new container
                sh '''
                docker pull ${DOCKER_IMAGE}:${DOCKER_TAG}
                docker run -d --name your-app-name -p 80:80 ${DOCKER_IMAGE}:${DOCKER_TAG}
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
