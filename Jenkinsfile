pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'image-analysis-app'
        DOCKER_TAG = 'latest'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                """
            }
        }
        
        stage('Deploy to Linode') {
            steps {
                sh """
                    # Stop existing container
                    docker stop ${DOCKER_IMAGE} || true
                    docker rm ${DOCKER_IMAGE} || true
                    
                    # Run new container
                    docker run -d \
                        --name ${DOCKER_IMAGE} \
                        --restart unless-stopped \
                        -p 8501:8501 \
                        ${DOCKER_IMAGE}:${DOCKER_TAG}
                    
                    # Reload Nginx
                    sudo systemctl reload nginx
                """
            }
        }
        
        stage('SSL Certificate Check') {
            steps {
                sh """
                    # Check SSL certificate expiry
                    certbot certificates
                """
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}