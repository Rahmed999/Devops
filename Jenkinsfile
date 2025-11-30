pipeline {
    agent any

    tools { 
        maven 'M2_HOME' // Make sure this matches your Jenkins Maven installation
    }

    environment {
        DOCKER_IMAGE = "ahmedridha92618/devopspipline/my-app:latest"
        DOCKER_REGISTRY = "docker.io"
        DOCKER_CREDENTIALS = "e0a06806-724b-42d2-9c5f-83a5d664075f" // Replace with your Jenkins Docker credentials ID
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
                sh 'mvn clean install'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ahmedridha92618/devopspipline/my-app:latest -f docker/Dockerfile .'


            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: "e0a06806-724b-42d2-9c5f-83a5d664075f", passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh "docker push ${DOCKER_IMAGE}"
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the logs!'
        }
    }
}
