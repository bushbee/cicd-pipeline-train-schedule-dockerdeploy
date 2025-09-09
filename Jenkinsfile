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
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production'
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    script {
                        def server = "${USERNAME}@${env.prod_ip}"
                        def image = "bushbee/train-schedule:${env.BUILD_NUMBER}"
        
                        // Pull latest image
                        sh "sshpass -p '$USERPASS' ssh -o StrictHostKeyChecking=no $server \"docker pull $image\""
        
                        // Stop and remove old container (ignore errors)
                        def stopStatus = sh(script: "sshpass -p '$USERPASS' ssh -o StrictHostKeyChecking=no $server \"docker stop train-schedule\"", returnStatus: true)
                        if (stopStatus != 0) {
                            echo "Container not running or stop failed"
                        }
        
                        def rmStatus = sh(script: "sshpass -p '$USERPASS' ssh -o StrictHostKeyChecking=no $server \"docker rm train-schedule\"", returnStatus: true)
                        if (rmStatus != 0) {
                            echo "Container removal failed or not found"
                        }
        
                        // Run new container
                        sh "sshpass -p '$USERPASS' ssh -o StrictHostKeyChecking=no $server \"docker run --restart always --name train-schedule -p 8080:8080 -d $image\""
                    }
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
