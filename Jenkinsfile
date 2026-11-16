
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

        stage('Build Image') {
            steps {
                withCredentials([
                    string(credentialsId: 'clinic-api-key', variable: 'API_KEY')
                ]) {
                    sh 'docker compose build'
                }
            }
        }

        stage('Deploy') {
            steps {
                withCredentials([
                    string(credentialsId: 'clinic-api-key', variable: 'API_KEY')
                ]) {
                    sh 'docker compose up -d'
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    docker compose ps
                    curl --fail --retry 5 --retry-delay 2 \
                      http://localhost:8000/docs -o /dev/null
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

