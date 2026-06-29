pipeline {
    agent any
    
    tools {
        nodejs 'NodeJS'
    }
    
    environment {
        IMAGE_NAME = "${env.BRANCH_NAME == 'main' ? 'nodemain' : 'nodedev'}"
        IMAGE_TAG  = 'v1.0'
        DOCKERHUB_REPO = 'dmugrazhina'
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
                sh "docker build -t ${DOCKERHUB_REPO}/${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }
        
        stage('Scan Docker Image for Vulnerabilities') {
            steps {
                script {
                    def vulnerabilities = sh(script: "trivy image --exit-code 0 --severity HIGH,MEDIUM,LOW --no-progress --cache-dir /tmp/trivy-cache-${env.BRANCH_NAME} ${DOCKERHUB_REPO}/${IMAGE_NAME}:${IMAGE_TAG}", returnStdout: true).trim()
                    echo "Vulnerability Report:\n${vulnerabilities}"
                }
            }
        }

stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                        docker push ${DOCKERHUB_REPO}/${IMAGE_NAME}:${IMAGE_TAG}
                        docker logout
                    """
                }
            }
        }
        stage('Trigger Deploy') {
            steps {
                build job: "${DEPLOY_JOB}", wait: false
            }
        }
    }
}