pipeline {

    agent any

    environment {
        DOCKER_IMAGE = "my-jenkins-app"
        DOCKER_TAG = "latest"
        DOCKER_REPO = "karthik449/my-jenkins-app"
        DOCKER_CREDENTIALS_ID = "docke-hub-credentials" // Jenkins credentials ID
        CONTAINER_NAME = "mycontainer1"
        CONTAINER_NAME1 = "mycontainer2"
    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/karthikeya964/docker-jenkins-app.git'
            }
        }

        stage('Docker Login') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD"
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                }
            }
        }

        stage('Run Container Locally') {
            steps {
                script {
                    sh """
                        # Stop and remove existing container if running
                        docker ps -a -q --filter name=${CONTAINER_NAME} | xargs -r docker stop || true
                        docker ps -a -q --filter name=${CONTAINER_NAME} | xargs -r docker rm || true

                        # Run new container
                        docker run -d -p 8092:80 --name ${CONTAINER_NAME} ${DOCKER_IMAGE}:${DOCKER_TAG}
                    """
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_REPO}:${DOCKER_TAG}"
                        sh "docker push ${DOCKER_REPO}:${DOCKER_TAG}"
                    }
                }
            }
        }

        stage('Deploy to Server') {
            steps {
                script {
                    sh """
                        sshpass -p "root" ssh -o StrictHostKeyChecking=no master@192.168.203.128 << EOF
                        docker pull ${DOCKER_REPO}:${DOCKER_TAG}
                        docker ps -a -q --filter name=${CONTAINER_NAME1} | xargs -r docker stop || true
                        docker ps -a -q --filter name=${CONTAINER_NAME1} | xargs -r docker rm || true
                        docker run -d -p 80:80 --name ${CONTAINER_NAME1} ${DOCKER_REPO}:${DOCKER_TAG}
                        EOF
                    """
                }
            }
        }

    }
}
