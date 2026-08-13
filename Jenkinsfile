pipeline {
    agent any

    environment {
        DOTNET_CLI_TELEMETRY_OPTOUT = '1'
        DOTNET_NOLOGO = '1'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Environment') {
            steps {
                sh '''
                    echo "===== .NET ====="
                    dotnet --version

                    echo "===== Git ====="
                    git --version

                    echo "===== Docker ====="
                    docker --version

                    echo "===== Docker Compose ====="
                    docker compose version
                '''
            }
        }

        stage('Restore') {
            steps {
                sh '''
                    dotnet restore ./src/eShop.AppHost/eShop.AppHost.csproj
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                    dotnet build ./src/eShop.AppHost/eShop.AppHost.csproj \
                        -c Release \
                        --no-restore
                '''
            }
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
            curl -f http://localhost:8085/
            curl -f http://localhost:8087/health
        '''
    }
}
    post {
        success {
            echo 'BUILD AND TEST SUCCESSFUL'
        }

        failure {
            echo 'PIPELINE FAILED'
        }
    }
}
