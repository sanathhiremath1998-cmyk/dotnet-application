pipeline {
    agent any

    environment {
        DOTNET_ROOT = "/opt/dotnet"
        PATH = "${DOTNET_ROOT}:${env.PATH}"
        COMPOSE_PROJECT_NAME = "jenkins-eshop"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Restore') {
            steps {
                sh '''
                    set -e
                    dotnet restore eShop.Web.slnf
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                    set -e
                    dotnet build eShop.Web.slnf \
                        --configuration Release \
                        --no-restore
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    set -e
                    dotnet test eShop.Web.slnf \
                        --configuration Release \
                        --no-build \
                        --logger "trx;LogFileName=test-results.trx"
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    set -e

                    echo "===== BUILDING DOCKER IMAGES ====="

                    docker build -t eshop-identity:test \
                        -f src/Identity.API/Dockerfile .

                    docker build -t eshop-catalog:test \
                        -f src/Catalog.API/Dockerfile .

                    docker build -t eshop-basket:test \
                        -f src/Basket.API/Dockerfile .

                    docker build -t eshop-ordering:test \
                        -f src/Ordering.API/Dockerfile .

                    docker build -t eshop-orderprocessor:test \
                        -f src/OrderProcessor/Dockerfile .

                    docker build -t eshop-paymentprocessor:test \
                        -f src/PaymentProcessor/Dockerfile .

                    docker build -t eshop-webhooks:test \
                        -f src/Webhooks.API/Dockerfile .

                    docker build -t eshop-webapp:test \
                        -f src/WebApp/Dockerfile .

                    docker build -t eshop-webhookclient:test \
                        -f src/WebhookClient/Dockerfile .

                    echo "===== DOCKER IMAGES ====="
                    docker images --format 'table {{.Repository}}\\t{{.Tag}}\\t{{.Size}}' | grep eshop-
                '''
            }
        }

        stage('Clean Old Compose Project') {
            steps {
                sh '''
                    set +e

                    echo "===== REMOVING OLD eshop-ci PROJECT ====="

                    docker compose -p eshop-ci down --remove-orphans

                    echo "===== VERIFYING OLD PROJECT ====="

                    docker compose ls
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    set -e

                    echo "===== DEPLOYING jenkins-eshop ====="

                    docker compose -p jenkins-eshop up -d --remove-orphans

                    echo "===== CONTAINERS ====="

                    docker compose -p jenkins-eshop ps
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    set -e

                    echo "===== WAITING FOR SERVICES ====="
                    sleep 15

                    echo "===== WEBAPP HEALTH CHECK ====="
                    curl -fsS -o /dev/null http://localhost:8185/
                    echo "WebApp OK"

                    echo "===== CATALOG HEALTH CHECK ====="
                    curl -fsS http://localhost:8187/health
                    echo
                    echo "Catalog API OK"

                    echo "===== VERIFYING CONTAINERS ====="

                    docker compose -p jenkins-eshop ps

                    echo "===== HEALTH CHECK PASSED ====="
                '''
            }
        }
    }

    post {

        always {
            echo 'Pipeline finished.'
        }

        success {
            echo 'Build, tests, Docker images and deployment succeeded.'
        }

        failure {
            echo 'Pipeline failed — check the logs above.'
        }
    }
}
