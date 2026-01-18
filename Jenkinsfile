pipeline {
    agent { label 'docker-agent' } 
    environment {
        DOCKER_HUB_USER = 'yourusername'
        IMAGE_NAME      = 'java-app'
        REGISTRY        = "${DOCKER_HUB_USER}/${IMAGE_NAME}"
    }
    stages {
        stage('Checkout SCM') {
            steps { checkout scm }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${REGISTRY}:${BUILD_NUMBER} ."
                    sh "docker tag ${REGISTRY}:${BUILD_NUMBER} ${REGISTRY}:latest"
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', 
                                                passwordVariable: 'DOCKER_HUB_PASSWORD', 
                                                usernameVariable: 'DOCKER_HUB_USERNAME')]) {
                    sh "echo ${DOCKER_HUB_PASSWORD} | docker login -u ${DOCKER_HUB_USERNAME} --password-stdin"
                    sh "docker push ${REGISTRY}:${BUILD_NUMBER}"
                    sh "docker push ${REGISTRY}:latest"
                }
            }
        }
        stage('Cleanup') {
            steps {
                sh "docker rmi ${REGISTRY}:${BUILD_NUMBER} ${REGISTRY}:latest"
                sh "docker image prune -f"
            }
        }
    }
}

