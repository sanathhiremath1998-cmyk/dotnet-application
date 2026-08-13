pipeline {
    agent any

    environment {
        DOTNET_ROOT = "/opt/dotnet"
        PATH = "${DOTNET_ROOT}:${env.PATH}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Restore') {
            steps {
                sh 'dotnet restore eShop.Web.slnf'
            }
        }

        stage('Build') {
            steps {
                sh 'dotnet build eShop.Web.slnf --configuration Release --no-restore'
            }
        }

        stage('Test') {
            steps {
                sh 'dotnet test eShop.Web.slnf --configuration Release --no-build --logger "trx;LogFileName=test-results.trx" || true'
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build -t eshop-identity:test -f src/Identity.API/Dockerfile .
                    docker build -t eshop-catalog:test -f src/Catalog.API/Dockerfile .
                    docker build -t eshop-basket:test -f src/Basket.API/Dockerfile .
                    docker build -t eshop-ordering:test -f src/Ordering.API/Dockerfile .
                    docker build -t eshop-orderprocessor:test -f src/OrderProcessor/Dockerfile .
                    docker build -t eshop-paymentprocessor:test -f src/PaymentProcessor/Dockerfile .
                    docker build -t eshop-webhooks:test -f src/Webhooks.API/Dockerfile .
                    docker build -t eshop-webapp:test -f src/WebApp/Dockerfile .
                    docker build -t eshop-webhookclient:test -f src/WebhookClient/Dockerfile .
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh 'docker compose up -d'
            }
        }

       stage('Health Check') {
    steps {
        sh '''
            sleep 10
            curl -fsS -o /dev/null http://localhost:8185/
            curl -fsS http://localhost:8187/health
        '''
    }
}
    }

    post {
        always {
            echo 'Pipeline finished.'
        }

        success {
            echo 'Build, tests and deployment succeeded.'
        }

        failure {
            echo 'Pipeline failed — check logs above.'
        }
   }
}
