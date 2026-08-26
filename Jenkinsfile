pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Image') {
            steps {
                sh '''
                    cp app.py /home/mbrar/app/app.py
                    cp requirements.txt /home/mbrar/app/requirements.txt
                    docker build -t jenkins-ci-lab:$BUILD_NUMBER /home/mbrar/app
                '''
            }
        }

        stage('Record Deployment Metadata') {
            steps {
                sh '''
                    [ -f /home/mbrar/current.txt ] && cp /home/mbrar/current.txt /home/mbrar/previous.txt
                    echo $BUILD_NUMBER > /home/mbrar/current.txt
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    docker rm -f flask-app || true
                    docker run -d --name flask-app --network app-net --restart unless-stopped jenkins-ci-lab:$BUILD_NUMBER
                    docker restart nginx-proxy
                    sleep 3
                '''
            }
        }

        stage('Health Check') {
            steps {
                script {
                    def healthy = sh(script: 'curl -f -s http://localhost/health', returnStatus: true) == 0
                    if (!healthy) {
                        echo "Health check FAILED for build $BUILD_NUMBER. Triggering automatic rollback..."
                        sh '/home/mbrar/rollback.sh'
                        sleep 3
                        sh 'curl -f http://localhost/health'
                        error("Deployment of build $BUILD_NUMBER failed health check. Automatically rolled back to previous version.")
                    }
                }
            }
        }

        stage('Backup') {
            steps {
                sh '/home/mbrar/backup.sh'
            }
        }
    }
}
