pipeline {
    agent any

    environment {
        DOCKER_HUB = 'tawfiqeleiba'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        SERVICES = "cart-service order-service payment-service product-service user-service"
    }

    stages {

        stage('Run Tests') {
            steps {
                script {
                    for (s in SERVICES.split()) {
                        dir("services/${s}") {
                            sh "npm install"
                            sh "npm run test --if-present"
                        }
                    }
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    def builds = [:]

                    for (s in SERVICES.split()) {
                        builds[s] = {
                            sh """
                            docker build \
                            -t ${DOCKER_HUB}/automated-e-commerce-${s}:${IMAGE_TAG} \
                            -t ${DOCKER_HUB}/automated-e-commerce-${s}:latest \
                            ./services/${s}
                            """
                        }
                    }

                    parallel builds
                }
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-cred',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                }
            }
        }

        stage('Push Images') {
            steps {
                script {
                    def pushes = [:]

                    for (s in SERVICES.split()) {
                        pushes[s] = {
                            sh "docker push ${DOCKER_HUB}/automated-e-commerce-${s}:${IMAGE_TAG}"
                            sh "docker push ${DOCKER_HUB}/automated-e-commerce-${s}:latest"
                        }
                    }

                    parallel pushes
                }
            }
        }

        // 🔥 (اختياري) Deploy باستخدام Docker Compose
        stage('Deploy') {
            steps {
                sh """
                docker-compose down || true
                docker-compose pull
                docker-compose up -d
                """
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully!'

            script {
                for (s in SERVICES.split()) {
                    sh "docker rmi ${DOCKER_HUB}/automated-e-commerce-${s}:${IMAGE_TAG} || true"
                }
            }
        }

        failure {
            echo '❌ Pipeline failed!'
        }
    }
}