pipeline {
    agent any
    environment {
        // Define environment variables here if needed
        DOCKER_IMAGE = 'your-docker-image'
        DOCKER_TAG = 'latest'
        K8S_DEPLOYMENT_NAME = 'your-deployment'
        K8S_NAMESPACE = 'default'
    }
    stages {
        stage('Checkout') {
            steps {
                // Checkout source code from GitHub
                git 'https://github.com/your-repo/your-project.git'
            }
        }
        stage('Build') {
            steps {
                // Build the application using Maven
                sh 'mvn clean package'
            }
        }
        stage('Publish') {
            steps {
                script {
                    // Build Docker image
                    sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                    // Push Docker image to Docker Hub or other registry
                    withDockerRegistry([credentialsId: 'dockerhub-credentials', url: 'https://index.docker.io/v1/']) {
                        sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    // Apply Kubernetes deployment
                    sh """
                    kubectl set image deployment/${K8S_DEPLOYMENT_NAME} ${K8S_DEPLOYMENT_NAME}=${DOCKER_IMAGE}:${DOCKER_TAG} --namespace=${K8S_NAMESPACE}
                    kubectl rollout status deployment/${K8S_DEPLOYMENT_NAME} --namespace=${K8S_NAMESPACE}
                    """
                }
            }
        }
    }
    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
