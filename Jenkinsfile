// Jenkinsfile
pipeline {
    agent {
        dockerContainer {
            image 'docker:dind'
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    environment {
        // Replace with your Docker Hub username
        DOCKER_HUB_USERNAME = 'your-dockerhub-username'
        // Create a unique tag for your image using the Jenkins build number
        IMAGE_NAME = "my-app"
        IMAGE_TAG = "${DOCKER_HUB_USERNAME}/${IMAGE_NAME}:${env.BUILD_NUMBER}"
        // The name of your container on the remote server
        CONTAINER_NAME = "my-app-container"
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out the code...'
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo 'Building the application...'
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                echo "Building Docker image: ${IMAGE_TAG}"
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh "docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}"
                        sh "docker build -t ${IMAGE_TAG} ."
                        echo "Pushing Docker image to Docker Hub..."
                        sh "docker push ${IMAGE_TAG}"
                    }
                }
            }
        }
        
        stage('Deploy via SSH') {
            steps {
                echo "Deploying container to Azure worker node..."
                sshagent(['azure-ssh-key']) {
                    sh """
                    // Put the public IP address of your Azure worker node here
                    REMOTE_HOST="<your-worker-node-ip>" 

                    # Stop and remove any existing container to prevent conflicts
                    ssh -o StrictHostKeyChecking=no ${REMOTE_HOST} "docker stop ${CONTAINER_NAME} || true"
                    ssh -o StrictHostKeyChecking=no ${REMOTE_HOST} "docker rm ${CONTAINER_NAME} || true"

                    # Pull the new image and run a new container
                    ssh -o StrictHostKeyChecking=no ${REMOTE_HOST} "docker pull ${IMAGE_TAG}"
                    ssh -o StrictHostKeyChecking=no ${REMOTE_HOST} "docker run -d --name ${CONTAINER_NAME} -p 80:80 ${IMAGE_TAG}"
                    """
                }
            }
        }
    }
}
