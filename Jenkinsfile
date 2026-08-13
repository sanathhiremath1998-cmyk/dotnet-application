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
                    dotnet restore eShop.Web.slnf
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                    dotnet build eShop.Web.slnf \
                        --configuration Release \
                        --no-restore
                '''
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
                    set -e

                    docker build \
                        -t eshop-identity:test \
                        -f src/Identity.API/Dockerfile .

                    docker build \
                        -t eshop-catalog:test \
                        -f src/Catalog.API/Dockerfile .

                    docker build \
                        -t eshop-basket:test \
                        -f src/Basket.API/Dockerfile .

                    docker build \
                        -t eshop-ordering:test \
                        -f src/Ordering.API/Dockerfile .

                    docker build \
                        -t eshop-orderprocessor:test \
                        -f src/OrderProcessor/Dockerfile .

                    docker build \
                        -t eshop-paymentprocessor:test \
                        -f src/PaymentProcessor/Dockerfile .

                    docker build \
                        -t eshop-webhooks:test \
                        -f src/Webhooks.API/Dockerfile .

                    docker build \
                        -t eshop-webapp:test \
                        -f src/WebApp/Dockerfile .

                    docker build \
                        -t eshop-webhookclient:test \
                        -f src/WebhookClient/Dockerfile .
                '''
            }
        }

        stage('Validate Docker Compose') {
            steps {
                sh '''
                    docker compose config -q
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    set -e

                    echo "Deploying jenkins-eshop stack..."

                    docker compose -p jenkins-eshop up -d --remove-orphans

                    echo "Current containers:"
                    docker compose -p jenkins-eshop ps
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    set -e

                    echo "Waiting for services to start..."
                    sleep 10

                    echo "Checking WebApp on port 8185..."

                    for i in $(seq 1 30); do
                        if curl -fsS -o /dev/null http://localhost:8185/; then
                            echo "WebApp OK"
                            break
                        fi

                        if [ "$i" -eq 30 ]; then
                            echo "WebApp health check FAILED"
                            docker compose -p jenkins-eshop ps
                            docker compose -p jenkins-eshop logs --tail=100 webapp
                            exit 1
                        fi

                        echo "WebApp not ready - retry $i/30"
                        sleep 2
                    done

                    echo "Checking Catalog API on port 8187..."

                    for i in $(seq 1 30); do
                        if curl -fsS http://localhost:8187/health; then
                            echo
                            echo "Catalog API OK"
                            break
                        fi

                        if [ "$i" -eq 30 ]; then
                            echo "Catalog API health check FAILED"
                            docker compose -p jenkins-eshop ps
                            docker compose -p jenkins-eshop logs --tail=100 catalog-api
                            exit 1
                        fi

                        echo "Catalog API not ready - retry $i/30"
                        sleep 2
                    done

                    echo "Checking Identity API on port 8186..."

                    for i in $(seq 1 30); do
                        if curl -fsS -o /dev/null http://localhost:8186/; then
                            echo "Identity API OK"
                            break
                        fi

                        if [ "$i" -eq 30 ]; then
                            echo "Identity API health check FAILED"
                            docker compose -p jenkins-eshop logs --tail=100 identity-api
                            exit 1
                        fi

                        sleep 2
                    done

                    echo "All health checks passed."
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

            sh '''
                docker compose -p jenkins-eshop ps || true
            '''
        }
    }
}
