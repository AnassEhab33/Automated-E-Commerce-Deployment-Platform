pipeline {
    agent any

    environment {
        DOCKER_HUB = 'tawfiqeleiba'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        SERVICES = "cart-service order-service payment-service product-service user-service"
    }

    stages {

        // stage('Clone') {
        //     steps {
        //         git 'https://github.com/AnassEhab33/Automated-E-Commerce-Deployment-Platform.git'
        //     }
        // }

        stage('Run Tests') {
            steps {
                script {
                    for (s in SERVICES.split()) {
                        sh "cd services/${s} && npm install && npm test || exit 1"
                    }
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    for (s in SERVICES.split()) {
                        sh """
                        docker build \
                        -t ${DOCKER_HUB}/automated-e-commerce-${s}:${IMAGE_TAG} \
                        -t ${DOCKER_HUB}/automated-e-commerce-${s}:latest \
                        ./services/${s}
                        """
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

        stage('Push Images') {
            steps {
                script {
                    for (s in SERVICES.split()) {
                        sh "docker push ${DOCKER_HUB}/automated-e-commerce-${s}:${IMAGE_TAG}"
                        sh "docker push ${DOCKER_HUB}/automated-e-commerce-${s}:latest"
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Build, Test, and Push completed successfully!'
            sh """
//                 docker rmi ${DOCKER_HUB}/automated-e-commerce-cart:${IMAGE_TAG} || true
//                 docker rmi ${DOCKER_HUB}/automated-e-commerce-order:${IMAGE_TAG} || true
//                 docker rmi ${DOCKER_HUB}/automated-e-commerce-payment:${IMAGE_TAG} || true
//                 docker rmi ${DOCKER_HUB}/automated-e-commerce-product:${IMAGE_TAG} || true
//                 docker rmi ${DOCKER_HUB}/automated-e-commerce-user:${IMAGE_TAG} || true
//             """
        }
    }
}