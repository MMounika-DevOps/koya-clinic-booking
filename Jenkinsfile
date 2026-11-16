
        stage('Build and Deploy') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'clinic-api-key',
                        variable: 'API_KEY'
                    )
                ]) {
                    sh '''
                        set +x

                        if [ -z "$API_KEY" ]; then
                            echo "ERROR: Jenkins API key is empty"
                            exit 1
                        fi

                        export API_KEY

                        docker compose build
                        docker compose up -d
                    '''
                }
            }
        }

