pipeline {
    agent none

    environment {
        DOCKERHUB_USER = 'shaikhirfan82'
        IMAGE_NAME = 'flask-app'
    }

    stages {

        stage('Checkout Code') {
            agent any
            steps {
                git branch: 'master',
                    url: 'https://github.com/shaikhirfan82/flask-again.git'

                stash name: 'source_code', includes: '**'
            }
        }

        stage('Build Docker Image') {
            agent { label 'build-node' }
            environment {
                DOCKERHUB_PASS = credentials('dockerhub-pass')
            }
            steps {
                unstash 'source_code'
                sh '''
                echo "$DOCKERHUB_PASS" | docker login -u $DOCKERHUB_USER --password-stdin
                docker build -t $DOCKERHUB_USER/$IMAGE_NAME:latest .
                docker push $DOCKERHUB_USER/$IMAGE_NAME:latest
                '''
            }
        }

        stage('Deploy') {
            agent { label 'deploy-node' }
            environment {
                DOCKERHUB_PASS = credentials('dockerhub-pass')
            }
            steps {
                sh '''
                echo "$DOCKERHUB_PASS" | docker login -u $DOCKERHUB_USER --password-stdin

                docker pull $DOCKERHUB_USER/$IMAGE_NAME:latest

                docker stop flask-app || true
                docker rm flask-app || true

                docker run -d \
                  --name flask-app \
                  -p 5000:5000 \
                  $DOCKERHUB_USER/$IMAGE_NAME:latest
                '''
            }
        }
    }
}
