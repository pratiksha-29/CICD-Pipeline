pipeline {
    agent any

    environment {
        IMAGE_NAME = "simple-app"
        IMAGE_TAG = "1.0"
        SONAR_HOST_URL = "http://3.110.48.47:9000" // Replace with your SonarQube IP
        SONAR_AUTH_TOKEN = credentials('sonar-token') // Jenkins Secret Text credential
        DOCKERHUB_USER = credentials('dockerhub-username') // Optional if pushing to DockerHub
        DOCKERHUB_PASS = credentials('dockerhub-password') // Optional if pushing to DockerHub
    }

    stages {

        // ----------------------------
        stage('Checkout') {
            steps {
                echo "1️⃣ Pulling latest code from GitHub repository"
                git branch: 'main', url: 'https://github.com/pratiksha-29/CICD-Pipeline/'
            }
        }

        // ----------------------------
        stage('Maven Build') {
            agent { docker { image 'maven:3.9.6-eclipse-temurin-17' } } // Maven + JDK17 container
            steps {
                echo "2️⃣ Building Java project using Maven"
                sh 'mvn clean verify -DskipTests'
            }
        }

        // ----------------------------
        stage('SonarQube Analysis') {
            agent { docker { image 'maven:3.9.6-eclipse-temurin-17' } }
            steps {
                echo "3️⃣ Running SonarQube analysis"
                sh """
                    mvn sonar:sonar \
                      -Dsonar.projectKey=simple-app \
                      -Dsonar.projectName=simple-app \
                      -Dsonar.host.url=$SONAR_HOST_URL \
                      -Dsonar.login=$SONAR_AUTH_TOKEN
                """
            }
        }

        // ----------------------------
        stage('Quality Gate') {
            steps {
                echo "4️⃣ Checking SonarQube Quality Gate"
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        // ----------------------------
        stage('Docker Build & Push') {
            agent { docker { image 'docker:24.0.2-dind' } } // Docker-in-Docker
            steps {
                echo "5️⃣ Building Docker image"
                sh """
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                    # Optional: Push to DockerHub
                    echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin
                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} $DOCKERHUB_USER/${IMAGE_NAME}:${IMAGE_TAG}
                    docker push $DOCKERHUB_USER/${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }

        // ----------------------------
        stage('Deploy') {
            agent { docker { image 'docker:24.0.2-dind' } }
            steps {
                echo "6️⃣ Deploying Docker container locally"
                sh """
                    docker rm -f ${IMAGE_NAME} || true
                    docker run -d --name ${IMAGE_NAME} -p 8080:8080 ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }
    }

    post {
        always {
            echo "🔹 Cleaning up workspace"
            cleanWs()
        }
        success {
            echo "✅ Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed! Check logs for details."
        }
    }
}
