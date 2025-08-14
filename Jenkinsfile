// Jenkinsfile for a containerd/Kubernetes environment
pipeline {
    // We are now using a Kubernetes agent.
    // This will instruct the Jenkins Kubernetes plugin to
    // dynamically create a Pod to run the pipeline.
    agent {
        kubernetes {
            // Define a Pod that has the tools we need for our pipeline.
            // This Pod will have a 'docker:dind' container inside it,
            // which provides the Docker daemon and CLI for our build.
            defaultContainer 'build-container'
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: build-container
    image: docker:dind
    command: ['cat']
    tty: true
    # This is a key part: we mount an empty directory as a volume
    # for the Docker daemon's storage, ensuring it works correctly.
    volumeMounts:
    - name: docker-graph-storage
      mountPath: /var/lib/docker
  volumes:
  - name: docker-graph-storage
    emptyDir: {}
'''
        }
    }

    environment {
        // Your correct Docker Hub username
        DOCKER_HUB_USERNAME = 'mefilmstudio'
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

        stage('Build and Push Docker Image') {
            steps {
                // The build-container from the Kubernetes agent definition
                // now has the `docker` command available.
                container('build-container') {
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
