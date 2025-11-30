pipeline {
    agent any

    environment {
        IMAGE_NAME = "simple-app"
        IMAGE_TAG = "1.0"

        // SonarQube URL
        SONAR_HOST_URL = "http://3.110.48.47:9000"

        // Jenkins credential ID for SonarQube token
        SONAR_TOKEN = credentials('sonar-token')
    }

    stages {

        stage('Checkout') {
            steps {
                // Checkout repo into workspace root
                git branch: 'main', url: 'https://github.com/pratiksha-29/CICD-Pipeline/'
            }
        }
         stage('Debug Workspace') {
            steps {
                sh "ls -l ${env.WORKSPACE}"
                sh "pwd"
            }
        }


        stage('Maven Build') {
    steps {
        sh "mvn clean verify -DskipTests"
    }
}

       

       stage('SonarQube Analysis') {
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

        stage('Docker Build') {
            steps {
                // Build Docker image from workspace root
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ${env.WORKSPACE}"
            }
        }

        stage('Deploy') {
            steps {
                sh """
                    # Remove existing container if any
                    docker rm -f ${IMAGE_NAME} || true

                    # Run the new container
                    docker run -d --name ${IMAGE_NAME} -p 8080:8080 ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }
    }

    post {
        always {
            cleanWs() // clean workspace after build
        }
    }
}
