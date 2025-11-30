pipeline {
    // Use any available Jenkins agent (can be master or a node)
    agent any

    // ----------------------------
    // Define environment variables here
    environment {
        // IMAGE_NAME: Name of the Docker image we will build locally
        IMAGE_NAME = "simple-app"

        // IMAGE_TAG: Version tag for Docker image
        IMAGE_TAG = "1.0"

        // SONAR_HOST_URL: URL of your SonarQube server
        SONAR_HOST_URL = "http://3.110.48.47:9000" // Replace <sonarqube-ip> with actual IP

        // SONAR_AUTH_TOKEN: Fetch the secret token from Jenkins credentials
        // You must create a Secret Text credential in Jenkins with ID = 'sonar-token'
        SONAR_AUTH_TOKEN = credentials('sonar-token')
    }

    stages {

        // ----------------------------
        stage('Checkout') {
            steps {
                echo "1️⃣ Pulling latest code from GitHub repository"
                // Git plugin will clone your repo
                git branch: 'main', url: 'https://github.com/pratiksha-29/CICD-Pipeline/'
            }
        }

        // ----------------------------
        stage('Maven Build') {
            steps {
                echo "2️⃣ Building Java project using Maven"
                // Build the project; skip tests to save time
                // Maven must be installed on Jenkins agent or Docker image
                sh 'mvn clean verify -DskipTests'
            }
        }

        // ----------------------------
        stage('SonarQube Analysis') {
            steps {
                echo "3️⃣ Running SonarQube analysis"
                // withSonarQubeEnv fetches the server details you configured in Jenkins
                withSonarQubeEnv('sonar-server') {
                    sh """
                        mvn sonar:sonar \
                        -Dsonar.projectKey=simple-app \       // Unique key in SonarQube
                        -Dsonar.projectName=simple-app \      // Display name in SonarQube
                        -Dsonar.host.url=$SONAR_HOST_URL \    // URL of SonarQube server
                        -Dsonar.login=$SONAR_AUTH_TOKEN       // Authentication token (secure)
                    """
                }
            }
        }

        // ----------------------------
        stage('Quality Gate') {
            steps {
                echo "4️⃣ Checking SonarQube Quality Gate"
                // Waits for SonarQube analysis results
                // If Quality Gate fails, pipeline aborts automatically
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        // ----------------------------
        stage('Docker Build') {
            steps {
                echo "5️⃣ Building Docker image locally"
                // Docker must be installed on the Jenkins host
                // Uses environment variables defined above
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                // Example expands to: docker build -t simple-app:1.0 .
            }
        }

        // ----------------------------
        stage('Deploy') {
            steps {
                echo "6️⃣ Deploying Docker container locally"
                sh '''
                # Remove old container if it exists
                docker rm -f ${IMAGE_NAME} || true

                # Run new container, map port 8080
                docker run -d --name ${IMAGE_NAME} -p 8080:8080 ${IMAGE_NAME}:${IMAGE_TAG}
                '''
                // Example expands to: docker run -d --name simple-app -p 8080:8080 simple-app:1.0
            }
        }
    }

    post {
        always {
            echo "🔹 Cleaning up workspace after build"
            cleanWs()  // Deletes cloned repo and temporary files
        }
        success {
            echo "✅ Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed! Check the logs for errors."
        }
    }
}
