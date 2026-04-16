pipeline {
    agent any 

    stages {
        stage('Build Microservices') {
            steps {
                script {
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
                    // تم تعطيل التست مؤقتاً حتى تقوم بإضافة سكريبتات الاختبار في ملفات الـ package.json
                    echo "Skipping tests for now..."
                }
            }
        }

        stage('Push to Artifactory') {
            steps {
                script {
                    // ملاحظة: تأكد أنك قمت بعمل docker login على سيرفر جنكنز
                    sh 'docker push tawfiqeleiba/automated-e-commerce-cart:1'
                    sh 'docker push tawfiqeleiba/automated-e-commerce-order:1'
                    sh 'docker push tawfiqeleiba/automated-e-commerce-payment:1'
                    sh 'docker push tawfiqeleiba/automated-e-commerce-product:1'
                    sh 'docker push tawfiqeleiba/automated-e-commerce-user:1'
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    sh 'docker-compose up -d'
                }
            }
        }
    }
}