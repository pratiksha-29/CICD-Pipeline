pipeline {
    agent { label 'docker-agent' } // runs on agent with Docker installed

    environment {
        IMAGE_NAME = "simple-app"
        IMAGE_TAG = "1.0"
        SONAR_HOST_URL = "http://3.110.48.47:9000"
        SONAR_TOKEN = credentials('sonar-token')
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/pratiksha-29/CICD-Pipeline/'
            }
        }

        stage('Maven Build') {
            steps {
                // Use Maven Docker container manually
                sh """
                docker run --rm \
                    -v ${env.WORKSPACE}:/app \
                    -w /app \
                    maven:3.9.6-eclipse-temurin-17 \
                    mvn clean verify -DskipTests
                """
            }
        }

        stage('SonarQube Analysis') {
            steps {
                sh """
                docker run --rm \
                    -v ${env.WORKSPACE}:/app \
                    -w /app \
                    maven:3.9.6-eclipse-temurin-17 \
                    mvn sonar:sonar \
                    -Dsonar.projectKey=simple-app \
                    -Dsonar.projectName=simple-app \
                    -Dsonar.host.url=$SONAR_HOST_URL \
                    -Dsonar.login=$SONAR_TOKEN
                """
            }
        }

        stage('Docker Build & Deploy') {
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
