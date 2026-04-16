pipeline {
    agent any 

    stages {
        stage('Build Docker Images') {
            steps {
                script {
                    echo "Building microservices images..."
                    // بناء الصور بناءً على أسماء الـ Repositories في حسابك على Docker Hub
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
                    // تشغيل التست لكل حاوية (Container) قبل الرفع
                    sh 'docker run --rm -e CI=true tawfiqeleiba/automated-e-commerce-cart:1 npm test'
                    sh 'docker run --rm -e CI=true tawfiqeleiba/automated-e-commerce-order:1 npm test'
                    sh 'docker run --rm tawfiqeleiba/automated-e-commerce-payment:1 pytest'
                    sh 'docker run --rm tawfiqeleiba/automated-e-commerce-product:1 pytest'
                    sh 'docker run --rm -e CI=true tawfiqeleiba/automated-e-commerce-user:1 npm test'
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    echo "Pushing images to tawfiqeleiba Docker Hub..."
                    // الرفع للحساب بتاعك الموضح في الصورة
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
            // تنظيف الجهاز عشان الـ Storage ما يتمليش في جينكنز
            sh 'docker image prune -f'
        }
    }
}