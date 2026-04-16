pipeline {
    agent any 

    stages {
        stage('Build Microservices') {
            steps {
                script {
                    // بناء الـ 5 صور بناءً على المسارات في مشروعك
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
                    // تشغيل التست لكل خدمة (بما أنها Node.js كما في الصورة)
                    sh 'docker run -e CI=true tawfiqeleiba/automated-e-commerce-cart:1 npm test'
                    // يمكنك إضافة بقية التستات هنا بنفس الطريقة
                }
            }
        }

        stage('Push to Artifactory') {
            steps {
                script {
                    // هنا يتم رفع الصور (تأكد من عمل docker login أولاً في سيرفر جنكنز)
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
                    // تشغيل المشروع بالكامل باستخدام docker-compose
                    sh 'docker-compose up -d'
                }
            }
        }
    }
}

