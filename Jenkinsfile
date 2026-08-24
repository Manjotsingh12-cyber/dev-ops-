pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Deploy') {
            steps {
                sh '''
                    cp app.py /home/mbrar/app/app.py
                    cp requirements.txt /home/mbrar/app/requirements.txt
                    /home/mbrar/deploy.sh $BUILD_NUMBER
                '''
            }
        }
        stage('Health Check') {
            steps {
                sh '''
                    sleep 3
                    curl -f http://localhost/health
                '''
            }
        }
    }
}
