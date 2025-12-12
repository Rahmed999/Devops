pipeline {
    agent any

    tools { 
        maven 'M2_HOME'   // Maven installation name in Jenkins
    }

    environment {
        DOCKER_CREDENTIALS = "e0a06806-724b-42d2-9c5f-83a5d664075f"
        IMAGE_TAG = "${env.GIT_COMMIT}"
        DOCKER_IMAGE = "ahmedridha92618/devopspipline:${IMAGE_TAG}"
        K8S_NAMESPACE = "devops"
    }

    stages {
        stage('Checkout Git') {
            steps {
                echo "Checking out Git repository..."
                git branch: 'main', url: 'https://github.com/Rahmed999/Devops.git'
            }
        }

        stage('Build with Maven') {
            steps {
                echo "Building project with Maven..."
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                sh "docker build -t ${DOCKER_IMAGE} -f docker/Dockerfile ."
            }
        }

        stage('Push Docker Image') {
            steps {
                echo "Pushing Docker image..."
                withCredentials([usernamePassword(
                    credentialsId: DOCKER_CREDENTIALS,
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    script {
                        int retries = 3
                        for (int i = 1; i <= retries; i++) {
                            try {
                                sh '''
                                    set -e
                                    export DOCKER_CLIENT_TIMEOUT=300
                                    export COMPOSE_HTTP_TIMEOUT=300
                                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                                    docker push ${DOCKER_IMAGE}
                                    docker logout
                                '''
                                echo "Docker push succeeded on attempt ${i}"
                                break
                            } catch (err) {
                                echo "Push attempt ${i} failed. Retrying..."
                                if (i == retries) {
                                    error("Docker push failed after ${retries} attempts")
                                }
                            }
                        }
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "Deploying to Kubernetes..."
                script {
                    // These paths are **relative to the Jenkins workspace**
                    sh """
                        # Create namespace if it doesn't exist
                        kubectl get ns ${K8S_NAMESPACE} || kubectl create ns ${K8S_NAMESPACE}

                        # Deploy MySQL
                        kubectl apply -f kub/mysql-deployment.yaml --namespace=${K8S_NAMESPACE} --validate=false

                        # Deploy Spring Boot app
                        kubectl apply -f kub/spring-deployment.yaml --namespace=${K8S_NAMESPACE} --validate=false

                        echo "Deployment finished!"
                        kubectl get pods --namespace=${K8S_NAMESPACE}
                        kubectl get svc --namespace=${K8S_NAMESPACE}
                    """
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
