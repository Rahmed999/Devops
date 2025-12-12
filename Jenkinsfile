pipeline {
    agent any

    tools { 
        maven 'M2_HOME'
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

        stage('Deploy MySQL to Kubernetes') {
            steps {
                script {
                    sh '''
                        echo "Deploying MySQL..."
                        kubectl apply -f kub/mysql-deployment.yaml --namespace=${K8S_NAMESPACE} --validate=false
                        echo "MySQL Deployment Finished!"
                        kubectl get pods --namespace=${K8S_NAMESPACE}
                        kubectl get svc --namespace=${K8S_NAMESPACE}
                    '''
                }
            }
        }

        stage('Deploy Spring Boot to Kubernetes') {
            steps {
                script {
                    sh '''
                        echo "Deploying Spring Boot App..."
                        kubectl apply -f kub/spring-deployment.yaml --namespace=${K8S_NAMESPACE} --validate=false
                        echo "Spring Boot Deployment Finished!"
                        kubectl get pods --namespace=${K8S_NAMESPACE}
                        kubectl get svc --namespace=${K8S_NAMESPACE}
                    '''
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
