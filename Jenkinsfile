pipeline {
    agent {
        docker {
            image 'python:3.11'
        }
    }
    stages {
        stage('Install dependencies') {
            steps {
                sh 'pip install -r requirements.txt'
            }
        }
        stage('Run tests') {
            steps {
                sh 'pytest --maxfail=1 --disable-warnings -q'
            }
        }
    }
}
