pipeline {
    agent any

    environment {
        DOTNET_CLI_TELEMETRY_OPTOUT = '1'
        DOTNET_NOLOGO = '1'
        ESHOP_USE_HTTP_ENDPOINTS = '1'
        APP_LOG = '/tmp/eshop-apphost.log'
        PID_FILE = '/tmp/eshop-apphost.pid'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Environment') {
            steps {
                sh '''
                    echo "=== .NET ==="
                    dotnet --version

                    echo "=== Docker ==="
                    docker --version

                    echo "=== Git ==="
                    git --version

                    echo "=== Docker Status ==="
                    docker ps
                '''
            }
        }

        stage('Restore') {
            steps {
                sh '''
                    dotnet restore \
                    ./src/eShop.AppHost/eShop.AppHost.csproj
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                    dotnet build \
                    ./src/eShop.AppHost/eShop.AppHost.csproj \
                    -c Release \
                    --no-restore
                '''
            }
        }

        stage('Stop Old Application') {
            steps {
                sh '''
                    if [ -f "$PID_FILE" ]; then
                        OLD_PID=$(cat "$PID_FILE")

                        if kill -0 "$OLD_PID" 2>/dev/null; then
                            echo "Stopping old AppHost: $OLD_PID"
                            kill "$OLD_PID" || true
                            sleep 10
                        fi

                        rm -f "$PID_FILE"
                    fi

                    docker ps
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    echo "Starting eShop Aspire AppHost..."

                    nohup dotnet run \
                    --project ./src/eShop.AppHost/eShop.AppHost.csproj \
                    -c Release \
                    --no-build \
                    > "$APP_LOG" 2>&1 &

                    echo $! > "$PID_FILE"

                    echo "PID: $(cat $PID_FILE)"

                    sleep 30
                '''
            }
        }

        stage('Verify') {
            steps {
                sh '''
                    echo "=== AppHost process ==="

                    PID=$(cat "$PID_FILE")

                    if ! kill -0 "$PID" 2>/dev/null; then
                        echo "eShop AppHost failed!"
                        cat "$APP_LOG"
                        exit 1
                    fi

                    echo "AppHost is running with PID $PID"

                    echo "=== Docker Containers ==="
                    docker ps

                    echo "=== Last AppHost Logs ==="
                    tail -100 "$APP_LOG"
                '''
            }
        }
    }

    post {
        success {
            echo 'eShop deployment successful.'
        }

        failure {
            echo 'eShop deployment failed.'

            sh '''
                echo "=== Application Log ==="
                tail -200 /tmp/eshop-apphost.log || true

                echo "=== Containers ==="
                docker ps -a || true
            '''
        }
    }
}
