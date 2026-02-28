pipeline {

    agent {
        docker {
            image 'maven:3.9-eclipse-temurin-17'
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    environment {
        IMAGE_NAME = "muhammedjaseel/pharmacy_app"
        IMAGE_TAG  = "v3"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Static Code Analysis') {
      environment {
        SONAR_URL = "http://172.20.10.2:9000"
      }
      steps {
        withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
          sh 'mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
        }
      }
    }
        stage('Build and Push Docker Image') {
      environment {
        // Update username or image name as per your requirements
        DOCKER_IMAGE = "muhammedjaseel/pharmacy_app:${IMAGE_TAG}"
        REGISTRY_CREDENTIALS = credentials('docker-cred')
      }
      steps {
        script {
            sh 'docker build -t ${DOCKER_IMAGE} .'
            def dockerImage = docker.image("${DOCKER_IMAGE}")
            docker.withRegistry('https://index.docker.io/v1/', "docker-cred") {
                dockerImage.push()
            }
        }
      }
    }
        stage('Run Container') {
            steps {
                sh '''
                docker stop pharmacy_app || true
                docker rm pharmacy_app || true

                docker run -d -p 4000:4040 \
                --name pharmacy_app \
                $IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }
    }
}
