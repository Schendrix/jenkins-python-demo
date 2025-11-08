pipeline {
    agent any

    environment {
        VENV = ".venv"
        APP_NAME = "jenkins-python-demo"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Schendrix/jenkins-python-demo.git'
            }
        }

        stage('Install System Packages') {
            steps {
                sh '''
                    # Make sure venv module and pip are available
                    apt update
                    apt install -y python3.13-venv python3-pip git
                '''
            }
        }

        stage('Setup Python Environment') {
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
                    pip install flake8
                    flake8 app.py tests/
                '''
            }
        }

        stage('Unit Tests') {
            steps {
                sh '''
                    . ${VENV}/bin/activate
                    export PYTHONPATH=$WORKSPACE
                    pytest -v --maxfail=1 --disable-warnings --junitxml=results.xml
                '''
            }
            post {
                always {
                    junit 'results.xml'  // Publish test results in Jenkins
                }
            }
        }

        stage('Deploy (Optional)') {
            when {
                branch 'main'
            }
            steps {
                echo "Deploying Flask app..."
                sh '''
                    . ${VENV}/bin/activate
                    nohup python3 app.py &
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline succeeded for branch: ${env.BRANCH_NAME}"
        }
        failure {
            echo "❌ Pipeline failed. Check linting or test reports."
        }
    }
}
