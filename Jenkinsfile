pipeline {
    agent any

    tools {
        maven 'M2_HOME'
    }

    environment {
        DOCKER_CREDENTIALS = "e0a06806-724b-42d2-9c5f-83a5d664075f"
        IMAGE_TAG = "${env.GIT_COMMIT}"
        NEXUS_HOST = "192.168.33.10:8085"
        NEXUS_REPO = "docker-repo"
        DOCKER_IMAGE = "${NEXUS_HOST}/${NEXUS_REPO}/student-app:${IMAGE_TAG}"
        K8S_NAMESPACE = "devops"
        KUBECONFIG = "/var/lib/jenkins/.kube/config"
        SONAR_PROJECT_KEY = "student-app"
        SONAR_PROJECT_NAME = "Student App"
        KUBECTL_CMD = "kubectl --kubeconfig=${KUBECONFIG}"
    }

    stages {
        stage('Checkout Git') {
            steps {
                git branch: 'main', url: 'https://github.com/Rahmed999/Devops.git'
            }
        }

        stage('Build & Unit Tests') {
            steps {
                sh 'mvn clean verify'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh """
                        mvn sonar:sonar \
                          -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                          -Dsonar.projectName="${SONAR_PROJECT_NAME}"
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                dir('docker') {
                    script {
                        docker.withRegistry("http://${NEXUS_HOST}", DOCKER_CREDENTIALS) {
                            def image = docker.build(DOCKER_IMAGE)
                            image.push()
                        }
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Ensure namespace exists
                    sh "${KUBECTL_CMD} get ns ${K8S_NAMESPACE} || ${KUBECTL_CMD} create ns ${K8S_NAMESPACE}"

                    // Apply all manifests at once
                    sh "${KUBECTL_CMD} apply -f kub/ -n ${K8S_NAMESPACE}"

                    // Update deployment image
                    sh "${KUBECTL_CMD} set image deployment/student-app student-app=${DOCKER_IMAGE} -n ${K8S_NAMESPACE} --record"

                    // Wait for rollout to complete
                    sh "${KUBECTL_CMD} rollout status deployment/student-app -n ${K8S_NAMESPACE}"
                }
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed!'
        }
    }
}
