// Example Jenkinsfile
pipeline {
    agent any // Run on any available Jenkins agent

    // Environment variables used throughout the pipeline
    environment {
        DOCKERHUB_USERNAME = 'mryashdoc'
        APP_NAME = 'my-app'
        DOCKERHUB_CREDS_ID = 'dockerhub-creds'
        KUBECONFIG_CREDS_ID = 'kubeconfig-creds'
    }

    stages {
        stage('1. Checkout Code') {
            steps {
                // Get code from Git
                git 'https://github.com/Mryash7803/demo-project'
            }
        }

        stage('2. Build Docker Image') {
            steps {
                script {
                    // Use the Git commit hash as the image tag for uniqueness
                    def imageTag = "${env.GIT_COMMIT.substring(0, 7)}"
                    // Define the full image name
                    env.DOCKER_IMAGE_NAME = "${DOCKERHUB_USERNAME}/${APP_NAME}:${imageTag}"
                    
                    // Build the image
                    docker.build(env.DOCKER_IMAGE_NAME, '.')
                }
            }
        }

        stage('3. Push Docker Image') {
            steps {
                // Log in to Docker Hub using the stored credentials
                docker.withRegistry('https://registry.hub.docker.com', DOCKERHUB_CREDS_ID) {
                    
                    // Push the image
                    docker.image(env.DOCKER_IMAGE_NAME).push()

                    // Also push a 'latest' tag
                    docker.image(env.DOCKER_IMAGE_NAME).push('latest')
                }
            }
        }

        stage('4. Deploy to Kubernetes') {
            steps {
                // Use the kubeconfig-creds ID from Jenkins credentials
                withKubeConfig([credentialsId: KUBECONFIG_CREDS_ID]) {
                    
                    // Use kubectl to update the image on the deployment
                    // This command finds the deployment and sets the new image
                    sh "kubectl set image deployment/my-app-deployment my-app=${env.DOCKER_IMAGE_NAME}"
                    
                    // Check the rollout status
                    sh "kubectl rollout status deployment/my-app-deployment"
                }
            }
        }
    }

    post {
        always {
            // Clean up the workspace
            cleanWs()
            
            // Clean up the local Docker image
            script {
                if (env.DOCKER_IMAGE_NAME) {
                    sh "docker rmi ${env.DOCKER_IMAGE_NAME}"
                    sh "docker rmi ${DOCKERHUB_USERNAME}/${APP_NAME}:latest"
                }
            }
        }
    }
}