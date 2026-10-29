pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'Source code checked out from GitHub.'
            }
        }

        stage('Run Unit Tests') {
            steps {
                sh '''
                    python3 -m venv .venv
                    . .venv/bin/activate
                    pip install -r requirements.txt
                    python -m unittest discover -s tests -p "test_app.py" -v
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t cicd-health-app:${BUILD_NUMBER} .'
            }
        }

        stage('Deploy Application') {
            steps {
                sh '''
                    docker rm -f cicd-health-app || true
                    docker run -d \
                      --name cicd-health-app \
                      -p 5000:5000 \
                      cicd-health-app:${BUILD_NUMBER}
                '''
            }
        }

        stage('Health Check and Rollback') {
            steps {
                sh '''
                    set +e
                    HEALTHY=0

                    for i in $(seq 1 10); do
                        if curl -fsS http://localhost:5000/health \
                            | grep -q '"status":"healthy"'; then
                            HEALTHY=1
                            break
                        fi
                        sleep 3
                    done

                    if [ "$HEALTHY" -eq 1 ]; then
                        echo "Health check passed."
                    else
                        echo "Health check failed. Starting rollback."
                        docker rm -f cicd-health-app || true
                        exit 1
                    fi
                '''
            }
        }
    }
}