pipeline {
    agent any

    environment {
        IMAGE_NAME = "simple-app"
        IMAGE_TAG = "1.0"

        // Correct SonarQube URL
        SONAR_HOST_URL = "http://3.110.48.47:9000"

        // Jenkins credential ID
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
                script {
                    // Path where pom.xml exists relative to workspace
                    def projectDir = ''  // e.g., '' if at root, or 'project' if in a subfolder

                    // Run Maven in Docker
                    sh """
                    docker run --rm \
                        -v ${env.WORKSPACE}:/app \
                        -w /app/${projectDir} \
                        maven:3.9.6-eclipse-temurin-17 \
                        mvn clean verify -DskipTests
                    """
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                sh """
                    docker run --rm \
                        -v "$PWD":/app \
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

        stage('Docker Build') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Deploy') {
            steps {
                sh """
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
