pipeline {
    agent {
        docker {
            image 'python:3.11'  // 使用 Python 3.11 官方 Docker 映像
        }
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install dependencies') {
            steps {
                sh 'pip install --upgrade pip'
                sh 'pip install -r requirements.txt'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'pytest --maxfail=1 --disable-warnings -q'
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
        }
        success {
            echo '✅ All tests passed successfully!'
        }
        failure {
            echo '❌ Some tests failed.'
        }
    }
}