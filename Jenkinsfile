pipeline {
    agent any

    environment {
        APP_DIR = '/home/ec2-user/projects/koya-clinic-booking'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Using the project checked out on EC2'
                sh 'git -C "$APP_DIR" status --short --branch'
            }
        }

        stage('Test') {
            steps {
                dir("${APP_DIR}") {
                    sh '''
                        docker run --rm \
                          -v "$PWD:/app" \
                          -w /app \
                          python:3.12-slim \
                          sh -c "pip install --no-cache-dir -r requirements.txt pytest httpx && pytest -q"
                    '''
                }
            }
        }

        stage('Build Image') {
            steps {
                dir("${APP_DIR}") {
                    sh 'docker compose build'
                }
            }
        }

        stage('Deploy') {
            steps {
                dir("${APP_DIR}") {
                    sh 'docker compose up -d'
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                dir("${APP_DIR}") {
                    sh '''
                        docker compose ps
                        curl --fail --retry 5 --retry-delay 2 \
                          http://localhost:8000/docs -o /dev/null
                    '''
                }
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
