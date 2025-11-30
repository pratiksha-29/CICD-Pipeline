pipeline {
    agent any

    environment {
        // Docker image name and tag
        IMAGE_NAME = "simple-app"
        IMAGE_TAG = "1.0"

        // SonarQube server URL
        SONAR_HOST_URL = "http://<SONARQUBE-IP>:9000"

        // SonarQube authentication token from Jenkins credentials
        SONAR_AUTH_TOKEN = credentials('sonar-token')
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
            steps {
                echo "2️⃣ Building Java project using Maven"
                sh 'mvn clean verify -DskipTests'
            }
        }

        // ----------------------------
        stage('SonarQube Analysis') {
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
        stage('Docker Build') {
            steps {
                echo "5️⃣ Building Docker image"
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        // ----------------------------
        stage('Deploy') {
            steps {
                echo "6️⃣ Deploying Docker container"
                sh """
                    docker rm -f ${IMAGE_NAME} || true
                    docker run -d --name ${IMAGE_NAME} -p 8080:8080 ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }
    }

    post {
        always {
            echo "🔹 Cleaning workspace"
            cleanWs()
        }
        success {
            echo "✅ Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed! Check logs."
        }
    }
}
