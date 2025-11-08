pipeline {
    agent any
    
    environment {
        APP_NAME = "jenkins-python-demo"
        VENV = ".venv"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "${env.BRANCH_NAME}", url: 'https://github.com/Schendrix/jenkins-python-demo.git'
            }
        }

        stage('Setup Environment') {
            steps {
                sh '''
                    python3 -m venv ${VENV}
                    . ${VENV}/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Lint') {
            steps {
                sh '''
                    . ${VENV}/bin/activate
                    flake8 app.py
                '''
            }
        }

        stage('Unit Tests') {
            steps {
                sh '''
                    . ${VENV}/bin/activate
                    pytest -v --maxfail=1 --disable-warnings --junitxml=results.xml
                '''
            }
            post {
                always {
                    junit 'results.xml'
                }
            }
        }

        stage('Deploy to Staging') {
            when {
                branch 'main'
            }
            steps {
                echo 'Deploying Flask app to staging server...'
                sh '''
                    nohup python3 app.py &
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Build and test succeeded for branch: ${env.BRANCH_NAME}"
        }
        failure {
            echo "❌ Pipeline failed."
        }
    }
}
