
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'Code checked out from GitHub'
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

                        if [ -z "$API_KEY" ]; then
                            echo "ERROR: Jenkins API key is empty"
                            exit 1
                        fi

                        printf 'API_KEY=%s\\n' "$API_KEY" > .env

                        if ! grep -q '^API_KEY=.' .env; then
                            echo "ERROR: Temporary environment file is invalid"
                            exit 1
                        fi

                        docker compose config --quiet
                        docker compose build
                        docker compose up -d
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    docker compose ps
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
            echo 'Pipeline failed. Check the stage logs.'
        }
    }
}

