pipeline {
  agent {
    docker {
        image 'muhammedjaseel/pharmacy_app:v1'
        args '--entrypoint=""'
    }
}
  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }
    stage('Build and Test') {
      steps {
        sh 'ls -ltr'
        // build the project and create a JAR file
        sh 'mvn clean package -DskipTests'
      }
    }
    stage('Static Code Analysis') {
      environment {
        SONAR_URL = "http://localhost:9000"
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
        DOCKER_IMAGE = "muhammedjaseel/pharmacy_app:${BUILD_NUMBER}"
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
    stage('Update Deployment File') {
        environment {
            GIT_REPO_NAME = "Pharmacy_story_CICD_complete"
            GIT_USER_NAME = "Muhammed-jaseel"
        }
        steps {
            withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                sh '''
                    git config user.email "your.email@example.com"
                    git config user.name "Muhammed Jaseel"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    
                    # Uncomment and update the lines below if you have Kubernetes manifests to update
                    # sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" Change/Deployment.yml
                    # git add Change/Deployment.yml
                    # git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    # git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                    
                    echo "Placeholder: Update deployment file step completed."
                '''
            }
        }
    }
  }
}
