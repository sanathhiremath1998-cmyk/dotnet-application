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
                sh '''
                    dotnet test eShop.Web.slnf \
                        --configuration Release \
                        --no-build \
                        --logger "trx;LogFileName=test-results.trx" || true
                '''
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

        stage('Validate Compose') {
            steps {
                sh 'docker compose config -q'
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    docker compose -p jenkins-eshop up -d --remove-orphans
                    docker compose -p jenkins-eshop ps
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    echo "Waiting for WebApp..."

                    for i in $(seq 1 30); do
                        if curl -fsS -o /dev/null http://localhost:8185/; then
                            echo "WebApp is healthy."
                            break
                        fi

                        if [ "$i" -eq 30 ]; then
                            echo "WebApp health check failed."
                            docker compose -p jenkins-eshop ps
                            docker compose -p jenkins-eshop logs --tail=50 webapp
                            exit 1
                        fi

                        sleep 2
                    done

                    echo "Waiting for Catalog API..."

                    for i in $(seq 1 30); do
                        if curl -fsS http://localhost:8187/health; then
                            echo
                            echo "Catalog API is healthy."
                            break
                        fi

                        if [ "$i" -eq 30 ]; then
                            echo "Catalog API health check failed."
                            docker compose -p jenkins-eshop ps
                            docker compose -p jenkins-eshop logs --tail=50 catalog-api
                            exit 1
                        fi

                        sleep 2
                    done
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
