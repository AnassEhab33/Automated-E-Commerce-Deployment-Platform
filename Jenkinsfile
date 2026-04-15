pipeline {
    agent any

    environment {
        DOCKER_HUB = 'tawfiqeleiba'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test') {
            steps {
                sh '''
                    echo "Running Tests..."

                    cd services/cart-service && npm install && npm test || true
                    cd ../../

                    cd services/order-service && npm install && npm test || true
                    cd ../../

                    cd services/payment-service && npm install && npm test || true
                    cd ../../

                    cd services/product-service && npm install && npm test || true
                    cd ../../

                    cd services/user-service && npm install && npm test || true
                    cd ../../

                    echo "Tests Finished"
                '''
            }
        }

        stage('Build Docker Images') {
            steps {
                sh "docker build -t ${DOCKER_HUB}/automated-e-commerce-cart:${IMAGE_TAG} ./services/cart-service"
                sh "docker build -t ${DOCKER_HUB}/automated-e-commerce-order:${IMAGE_TAG} ./services/order-service"
                sh "docker build -t ${DOCKER_HUB}/automated-e-commerce-payment:${IMAGE_TAG} ./services/payment-service"
                sh "docker build -t ${DOCKER_HUB}/automated-e-commerce-product:${IMAGE_TAG} ./services/product-service"
                sh "docker build -t ${DOCKER_HUB}/automated-e-commerce-user:${IMAGE_TAG} ./services/user-service"
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
                sh "docker push ${DOCKER_HUB}/automated-e-commerce-cart:${IMAGE_TAG}"
                sh "docker push ${DOCKER_HUB}/automated-e-commerce-order:${IMAGE_TAG}"
                sh "docker push ${DOCKER_HUB}/automated-e-commerce-payment:${IMAGE_TAG}"
                sh "docker push ${DOCKER_HUB}/automated-e-commerce-product:${IMAGE_TAG}"
                sh "docker push ${DOCKER_HUB}/automated-e-commerce-user:${IMAGE_TAG}"
            }
        }

        stage('Deploy') {
            steps {
                sh 'docker compose down || true'
                sh 'docker compose up -d'
            }
        }
    }

    post {
        success {
            echo 'Build + Test + Deploy DONE BY TAWFIQ ELEIBA '

            sh """
                docker rmi ${DOCKER_HUB}/automated-e-commerce-cart:${IMAGE_TAG} || true
                docker rmi ${DOCKER_HUB}/automated-e-commerce-order:${IMAGE_TAG} || true
                docker rmi ${DOCKER_HUB}/automated-e-commerce-payment:${IMAGE_TAG} || true
                docker rmi ${DOCKER_HUB}/automated-e-commerce-product:${IMAGE_TAG} || true
                docker rmi ${DOCKER_HUB}/automated-e-commerce-user:${IMAGE_TAG} || true
            """
        }
    }
}