pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE_NAME = 'admin-server'
        DOCKER_TAG = 'latest'
        APP_PORT = '9090'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build .jar') {
            agent {
                docker {
                    image 'maven:3.8.8-eclipse-temurin-17'
                    reuseNode true
                }
            }
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Build Docker Image') {
            agent {
                docker {
                    image 'docker:24.0.7'
                    args '''-v /var/run/docker.sock:/var/run/docker.sock
                           -v /tmp:/tmp
                           -v /var/lib/jenkins/.docker:/root/.docker
                           -u root'''
                    reuseNode true
                }
            }
            steps {
                script {
                    sh 'docker build -t ${DOCKER_IMAGE_NAME}:${DOCKER_TAG} .'
                }
            }
        }

        stage('Run Container') {
            steps {
                script {
                    // First remove any existing container
                    sh 'docker rm -f admin-server || true'
                    
                    // Run new container with proper port mapping
                    sh 'docker run -d -p ${APP_PORT}:${APP_PORT} --name admin-server ${DOCKER_IMAGE_NAME}:${DOCKER_TAG}'
                    
                    // Add health check
                    sh '''
                        echo "Waiting for Admin Server to start..."
                        while ! curl -s http://localhost:${APP_PORT}/actuator/health; do
                            sleep 5
                        done
                        echo "Admin Server is up!"
                    '''
                }
            }
        }
    }

    post {
        failure {
            sh 'docker rm -f admin-server || true'
        }
    }
}