pipeline {
    agent {
        docker {
            image 'python:3.11'  // 使用官方 Python 映像
            args '-u root'       // 用 root 執行 pipeline 內部命令
        }
    }

    stages {
        stage('Install dependencies') {
            steps {
                sh '''
                    pip install --upgrade pip
                    pip install -r requirements.txt
                    pip install pytest
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                    python --version
                    python3 --version
                    pytest --maxfail=1 --disable-warnings -q
                    python main.py
                    python3 main.py
                '''
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
            echo '❌ Tests failed.'
        }
    }
}
