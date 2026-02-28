pipeline {

    agent none

    environment {
        IMAGE_NAME = "muhammedjaseel/pharmacy_app"
        IMAGE_TAG  = "v3"
        SONAR_URL  = "http://172.20.10.2:9000"
    }

    stages {

        stage('Checkout') {
            agent any
            steps {
                checkout scm
            }
        }

        // ---------------- BUILD ----------------
        stage('Build & Test') {
            agent {
                docker {
                    image 'maven:3.9-eclipse-temurin-17'
                    args '--entrypoint=""'
                }
            }
            steps {
                sh 'mvn clean package'
            }
        }

        // ---------------- SONAR ----------------
        stage('Static Code Analysis') {
            agent {
                docker {
                    image 'maven:3.9-eclipse-temurin-17'
                    args '--entrypoint="" --add-host=host.docker.internal:host-gateway'
                }
            }
            steps {
                withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
                    sh """
                    mvn sonar:sonar \
                    -Dsonar.login=$SONAR_AUTH_TOKEN \
                    -Dsonar.host.url=$SONAR_URL
                    """
                }
            }
        }

        // ---------------- DOCKER BUILD ----------------
        stage('Build and Push Docker Image') {
            agent any
            environment {
                DOCKER_IMAGE = "${IMAGE_NAME}:${IMAGE_TAG}"
            }
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE} ."

                    docker.withRegistry('https://index.docker.io/v1/', 'docker-cred') {
                        docker.image("${DOCKER_IMAGE}").push()
                    }
                }
            }
        }

        // ---------------- RUN CONTAINER ----------------
        stage('Run Container') {
            agent any
            steps {
                sh """
                docker stop pharmacy_app || true
                docker rm pharmacy_app || true

                docker run -d -p 4000:4040 \
                --name pharmacy_app \
                ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }
    }
}
