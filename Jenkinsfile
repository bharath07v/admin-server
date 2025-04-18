pipeline {
	agent {
    	docker {
        	image 'maven:3.8.8-eclipse-temurin-17'
    	}
	}

    environment {
        // Optional: Define any environment variables here
        DOCKER_IMAGE_NAME = 'admin-server'
        DOCKER_TAG = 'latest'
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Build .jar') {
            steps {
                script {
                    // Run Maven to build the project and create the JAR file
                    sh 'mvn clean install'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image for the API Gateway
                    sh "docker build -t ${DOCKER_IMAGE_NAME}:${DOCKER_TAG} ."
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    // Run the Docker container
                    sh "docker run -d -p 9090:9090 --name admin-server-container ${DOCKER_IMAGE_NAME}:${DOCKER_TAG}"
                }
            }
        }
    }

    post {
        always {
            // Clean up any Docker containers or images if needed
            sh 'docker rm -f admin-server-container || true'
        }

        success {
            // Any actions after success (e.g., sending a success notification)
            echo 'Build and deployment successful!'
        }

        failure {
            // Actions if build fails (e.g., sending failure notification)
            echo 'Build failed. Please check logs.'
        }
    }
}
