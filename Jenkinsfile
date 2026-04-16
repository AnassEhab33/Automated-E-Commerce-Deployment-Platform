pipeline {
    agent any

    environment {
        DOCKER_HUB = 'tawfiqeleiba'
        IMAGE_TAG = "${env.BUILD_NUMBER}-${env.BRANCH_NAME?.replaceAll('/', '-')}"
        SERVICES = "cart-service order-service payment-service product-service user-service"
    }

    stages {

        stage('Run Tests') {
            steps {
                script {
                    for (s in SERVICES.split()) {
                        dir("services/${s}") {
                            sh "npm install --no-audit --no-fund"
                            sh "npm run test --if-present"
                        }
                    }
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

        stage('Build Docker Images') {
            steps {
                script {
                    def builds = [:]

                    for (s in SERVICES.split()) {
                        builds[s] = {
                            sh """
                            docker build --no-cache \
                            -t ${DOCKER_HUB}/automated-e-commerce-${s}:${IMAGE_TAG} \
                            -t ${DOCKER_HUB}/automated-e-commerce-${s}:latest \
                            ./services/${s}
                            """
                        }
                    }

                    parallel builds, failFast: true
                }
            }
        }

        stage('Push Images') {
            steps {
                script {
                    def pushes = [:]

                    for (s in SERVICES.split()) {
                        pushes[s] = {
                            sh """
                            docker push ${DOCKER_HUB}/automated-e-commerce-${s}:${IMAGE_TAG}
                            docker push ${DOCKER_HUB}/automated-e-commerce-${s}:latest
                            """
                        }
                    }

                    parallel pushes, failFast: true
                }
            }
        }

        stage('Deploy') {
            steps {
                sh """
                docker compose up -d --build
                """
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

        always {
            sh "docker logout || true"
        }
    }
}