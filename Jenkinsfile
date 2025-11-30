pipeline {
    agent any

    environment {
        IMAGE_NAME = "simple-app"
        IMAGE_TAG = "1.0"
        SONAR_HOST_URL = "http://3.110.48.47:9000"
        SONAR_AUTH_TOKEN = credentials('sonar-token')
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Pulling latest code"
                git branch: 'main', url: 'https://github.com/pratiksha-29/CICD-Pipeline/'
            }
        }

        stage('Maven Build') {
            steps {
                echo "Building Java project using Maven in Docker"
                sh 'docker run --rm -v $PWD:/app -w /app maven:3.9.6-eclipse-temurin-17 mvn clean verify -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo "Running SonarQube analysis in Docker"
                sh """
                    docker run --rm -v $PWD:/app -w /app maven:3.9.6-eclipse-temurin-17 \
                    mvn sonar:sonar \
                      -Dsonar.projectKey=simple-app \
                      -Dsonar.projectName=simple-app \
                      -Dsonar.host.url=$SONAR_HOST_URL \
                      -Dsonar.login=$SONAR_AUTH_TOKEN
                """
            }
        }

        stage('Docker Build & Deploy') {
            steps {
                echo "Building Docker image"
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."

                echo "Running Docker container"
                sh """
                    docker rm -f ${IMAGE_NAME} || true
                    docker run -d --name ${IMAGE_NAME} -p 8080:8080 ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }
    }

    post {
        always {
            echo "Cleaning workspace"
            cleanWs()
        }
    }
}
