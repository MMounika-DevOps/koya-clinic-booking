pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out Koya Clinic Booking'
                sh 'git status --short --branch'
            }
        }

        stage('Test') {
            steps {
                sh '''
                    docker run --rm \
                      -v "$WORKSPACE:/app" \
                      -w /app \
                      python:3.12-slim \
                      sh -c "pip install --no-cache-dir -r requirements.txt pytest httpx && pytest -q"
                '''
            }
        }

        stage('Build and Deploy') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'clinic-api-key',
                        variable: 'API_KEY'
                    )
                ]) {
                    sh '''
                        set -eu
                        set +x
                        umask 077

                        trap 'rm -f .env' EXIT

                        if [ -z "${API_KEY:-}" ]; then
                            echo "ERROR: Jenkins API key is empty or unavailable"
                            exit 1
                        fi

                        printf 'API_KEY=%s\\n' "$API_KEY" > .env

                        if ! grep -q '^API_KEY=.' .env; then
                            echo "ERROR: Temporary environment file is invalid"
                            exit 1
                        fi

                        echo "Validating Docker Compose configuration"
                        docker compose config --quiet

                        echo "Building Docker image"
                        docker compose build

                        echo "Deploying application"
                        docker compose up -d
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    set -eu

                    echo "Checking running container"
                    docker ps --filter name=koya-clinic-booking \
                      --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'

                    echo "Checking application endpoint"
                    curl --fail --retry 5 --retry-delay 2 \
                      http://localhost:8000/docs -o /dev/null

                    echo "Application health check passed"
                '''
            }
        }
    }

    post {
        success {
            echo 'Koya Clinic Booking deployment completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check Console Output for the error.'
        }

        always {
            sh 'rm -f .env'
        }
    }
}
