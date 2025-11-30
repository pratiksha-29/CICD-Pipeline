pipeline {
    agent none  // No global agent

    environment {
        IMAGE_NAME = "simple-app"
        IMAGE_TAG = "1.0"
        SONAR_HOST_URL = "http://3.110.48.47:9000"
        SONAR_TOKEN = credentials('sonar-token')
    }

    stages {
        stage('Checkout') {
            agent any  // Checkout on Jenkins host
            steps {
                git branch: 'main', url: 'https://github.com/pratiksha-29/CICD-Pipeline/'
            }
        }

        stage('Maven Build') {
            agent {
                docker {
                    image 'maven:3.9.6-eclipse-temurin-17'
                    args '-v /var/jenkins_home/.m2:/root/.m2'
                }
            }
            steps {
                sh 'mvn clean verify -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            agent any  // Run on host
            steps {
                sh """
                mvn sonar:sonar \
                    -Dsonar.projectKey=simple-app \
                    -Dsonar.projectName=simple-app \
                    -Dsonar.host.url=$SONAR_HOST_URL \
                    -Dsonar.login=$SONAR_TOKEN
                """
            }
        }

        stage('Docker Build & Deploy') {
            agent any  // Run on Jenkins host/container with Docker
            steps {
                sh """
                docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ${env.WORKSPACE}
                docker rm -f ${IMAGE_NAME} || true
                docker run -d --name ${IMAGE_NAME} -p 8080:8080 ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
