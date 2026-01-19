pipeline {
    agent { label 'docker-agent' } 
    environment {
        DOCKER_HUB_USER = 'yourusername'
        IMAGE_NAME      = 'java-app'
        REGISTRY        = "${DOCKER_HUB_USER}/${IMAGE_NAME}"
    }
    stages {
        stage('Checkout SCM') {
            // Stage 1: Pull the source code from the repository linked to the Jenkins job
            steps { checkout scm }
        }
        stage('Build Docker Image') {
            // Stage 2: Build the Docker image using the Dockerfile in the root directory
            steps {
                script {
                    sh "docker build -t ${REGISTRY}:${BUILD_NUMBER} ."
                    sh "docker tag ${REGISTRY}:${BUILD_NUMBER} ${REGISTRY}:latest"
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                // Securely injects credentials stored in Jenkins under the ID 'docker-hub-creds'
                withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', 
                                                passwordVariable: 'DOCKER_HUB_PASSWORD', 
                                                usernameVariable: 'DOCKER_HUB_USERNAME')]) {
                    // Securely log into Docker Hub using the injected environment variables
                    sh "echo ${DOCKER_HUB_PASSWORD} | docker login -u ${DOCKER_HUB_USERNAME} --password-stdin"
                    // Push both the versioned tag and the latest tag
                    sh "docker push ${REGISTRY}:${BUILD_NUMBER}"
                    sh "docker push ${REGISTRY}:latest"
                }
            }
        }
        stage('Cleanup') {
            steps {
                // Stage 4: Remove local images to save disk space on the build agent
               // Remove the specific images created during this build
                sh "docker rmi ${REGISTRY}:${BUILD_NUMBER} ${REGISTRY}:latest"
                // Force remove any dangling images (layers left behind)
                sh "docker image prune -f"
            }
        }
    }
}

