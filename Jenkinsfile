
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'Source code is available in the workspace.'
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
    }
}
