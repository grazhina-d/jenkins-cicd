pipeline {
    agent any
    
    tools {
        nodejs 'NodeJS'
    }
    
    environment {
        IMAGE_NAME = "${env.BRANCH_NAME == 'main' ? 'nodemain' : 'nodedev'}"
        IMAGE_TAG  = 'v1.0'
        PORT       = "${env.BRANCH_NAME == 'main' ? '3000' : '3001'}"
        CONTAINER_NAME = "${env.BRANCH_NAME == 'main' ? 'app-main' : 'app-dev'}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                sh 'npm install'
            }
        }
        
        stage('Test') {
            steps {
                sh 'npm test'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }
        
        stage('Deploy') {
            steps {
                sh """
                    docker rm -f ${CONTAINER_NAME} || true
                    docker run -d --name ${CONTAINER_NAME} -p ${PORT}:3000 ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }
    }
}