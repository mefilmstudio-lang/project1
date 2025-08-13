// Jenkinsfile
pipeline {
    agent any

    environment {
        // Replace with your Docker Hub username
        DOCKER_HUB_USERNAME = 'your-dockerhub-username'
        // Create a unique tag for your image using the Jenkins build number
        IMAGE_NAME = "my-app"
        IMAGE_TAG = "${DOCKER_HUB_USERNAME}/${IMAGE_NAME}:${env.BUILD_NUMBER}"
        DEPLOYMENT_NAME = "my-app-deployment"
        CONTAINER_NAME = "my-app-container"
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out the code...'
                // This step is automatic if you configure your job to use SCM
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo 'Building the application...'
                // Example build command (e.g., for a Node.js app)
                // sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
                // Example test command
                // sh 'npm test'
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                echo "Building Docker image: ${IMAGE_TAG}"
                script {
                    // Build the Docker image from the Dockerfile
                    sh "docker build -t ${IMAGE_TAG} ."
                    
                    // Use withCredentials to securely handle Docker Hub login
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh "docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}"
                        echo "Pushing Docker image to Docker Hub..."
                        sh "docker push ${IMAGE_TAG}"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "Deploying new image to Kubernetes..."
                // Use kubectl to update the deployment with the new image
                // This command tells Kubernetes to update the 'my-app-container' in 'my-app-deployment'
                // with the new image we just pushed.
                sh "kubectl set image deployment/${DEPLOYMENT_NAME} ${CONTAINER_NAME}=${IMAGE_TAG}"
            }
        }
    }
}
