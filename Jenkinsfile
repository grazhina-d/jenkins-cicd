pipeline {
    agent any

    environment {
        IMAGE_NAME = "${env.BRANCH_NAME == 'main' ? 'nodemain' : 'nodedev'}"
        IMAGE_TAG  = 'v1.0'
        DOCKERHUB_REPO = 'dmugrazhina'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Hadolint') {
            steps {
                sh 'hadolint Dockerfile || true'
            }
        }

        stage('Build') {
            agent {
                docker {
                    image 'node:7.8.0'
                    reuseNode true
                }
            }
            steps {
                sh 'npm install'
            }
        }

        stage('Test') {
            agent {
                docker {
                    image 'node:7.8.0'
                    reuseNode true
                }
            }
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
                script {
                    def deployJob = env.BRANCH_NAME == 'main' ? 'Deploy_to_main' : 'Deploy_to_dev'
                    build job: deployJob, wait: false
                }
            }
        }
    }
}

