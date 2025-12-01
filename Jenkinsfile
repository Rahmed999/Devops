pipeline {
    agent any

    tools { 
        maven 'M2_HOME'
    }

    environment {
        DOCKER_IMAGE = "ahmedridha92618/devopspipline:latest"
        DOCKER_CREDENTIALS = "e0a06806-724b-42d2-9c5f-83a5d664075f"
    }

    stages {
        stage('Checkout Git') {
            steps {
                git branch: 'main', 
                    url: 'https://github.com/Rahmed999/Devops.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${DOCKER_IMAGE} -f docker/Dockerfile .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: DOCKER_CREDENTIALS,
                    passwordVariable: 'DOCKER_PASS',
                    usernameVariable: 'DOCKER_USER'
                )]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh "docker push ${DOCKER_IMAGE}"
                }
            }
        }
    }

    post {
        success { echo 'Pipeline completed successfully!' }
        failure { echo 'Pipeline failed. Check the logs!' }
    }
}
