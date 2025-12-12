pipeline {
    agent any

    tools { 
        maven 'M2_HOME'
    }

    environment {
        DOCKER_CREDENTIALS = "e0a06806-724b-42d2-9c5f-83a5d664075f"

        IMAGE_TAG = "${env.GIT_COMMIT}"

        NEXUS_HOST = "192.168.33.10:8085"      // Nexus docker port
        NEXUS_REPO = "docker-repo"

        DOCKER_IMAGE = "${NEXUS_HOST}/${NEXUS_REPO}/student-app:${IMAGE_TAG}"

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

        stage('Push Docker Image to Nexus') {
            steps {
                echo "Pushing Docker image to Nexus..."

                withCredentials([usernamePassword(
                    credentialsId: DOCKER_CREDENTIALS,
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                   sh '''
                set -e
                echo "$DOCKER_PASS" | docker login 192.168.33.10:8085 -u "$DOCKER_USER" --password-stdin
                docker push ${DOCKER_IMAGE}
                docker logout 192.168.33.10:8085
            	'''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "Deploying to Kubernetes..."

                sh """
                    kubectl get ns ${K8S_NAMESPACE} || kubectl create ns ${K8S_NAMESPACE}

                    kubectl apply -f kub/mysql-deployment.yaml -n ${K8S_NAMESPACE}
                    kubectl apply -f kub/spring-deployment.yaml -n ${K8S_NAMESPACE}

                    # Update image to newest version from Nexus
                    kubectl set image deployment/student-app student-app=${DOCKER_IMAGE} \
                        -n ${K8S_NAMESPACE} --record

                    kubectl get pods -n ${K8S_NAMESPACE}
                    kubectl get svc -n ${K8S_NAMESPACE}
                """
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
