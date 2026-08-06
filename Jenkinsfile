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

    post {
        success {
            echo 'BUILD AND TEST SUCCESSFUL'
        }

        failure {
            echo 'PIPELINE FAILED'
        }
    }
}
