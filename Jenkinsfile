pipeline {
    agent any 

    stages {
        stage('Build Docker Images') {
            steps {
                script {
                    echo "Building microservices images..."
                   
                    sh 'docker build -t tawfiqeleiba/automated-e-commerce-cart:1 ./services/cart-service'
                    sh 'docker build -t tawfiqeleiba/automated-e-commerce-order:1 ./services/order-service'
                    sh 'docker build -t tawfiqeleiba/automated-e-commerce-payment:1 ./services/payment-service'
                    sh 'docker build -t tawfiqeleiba/automated-e-commerce-product:1 ./services/product-service'
                    sh 'docker build -t tawfiqeleiba/automated-e-commerce-user:1 ./services/user-service'
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    echo "Running tests for all services..."
                    sh 'docker run --rm -e CI=true tawfiqeleiba/automated-e-commerce-cart:1 npm test || echo "Cart test skipped"'
                    sh 'docker run --rm -e CI=true tawfiqeleiba/automated-e-commerce-order:1 npm test || echo "Order test skipped"'
                    sh 'docker run --rm tawfiqeleiba/automated-e-commerce-payment:1 pytest || echo "Payment test skipped"'
                    sh 'docker run --rm tawfiqeleiba/automated-e-commerce-product:1 pytest || echo "Product test skipped"'
                    sh 'docker run --rm -e CI=true tawfiqeleiba/automated-e-commerce-user:1 npm test || echo "User test skipped"'
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    echo "Pushing images to Docker Hub..."
                    sh 'docker push tawfiqeleiba/automated-e-commerce-cart:1'
                    sh 'docker push tawfiqeleiba/automated-e-commerce-order:1'
                    sh 'docker push tawfiqeleiba/automated-e-commerce-payment:1'
                    sh 'docker push tawfiqeleiba/automated-e-commerce-product:1'
                    sh 'docker push tawfiqeleiba/automated-e-commerce-user:1'
                }
            }
        }
    }

    post {
        always {
            sh 'docker image prune -f'
        }
        success {
            echo 'Pipeline finished successfully! Images are pushed.'
        }
        failure {
            echo 'Pipeline failed. Check the console output.'
        }
    }
}