pipeline {
    agent none

    environment {
        DOCKERHUB_USER = 'shivkumar1993docker'
        IMAGE_NAME = 'flask-app'
        DOCKERHUB_PASS = credentials('dockerhub-pass')
    }

    stages {

        stage('Checkout Code') {
            agent { label 'built-in' }
            steps {
                git branch: 'master',
                    url: 'https://github.com/shivmatkawala/flask-dind-project.git'
                
                stash name: 'source_code', includes: '**'
            }
        }

        stage('Build Docker Image') {
            agent { label 'build-node' }
            steps {
                unstash 'source_code'
                sh '''
                docker login -u $DOCKERHUB_USER -p $DOCKERHUB_PASS
                docker build -t $DOCKERHUB_USER/$IMAGE_NAME:latest .
                docker push $DOCKERHUB_USER/$IMAGE_NAME:latest
                '''
            }
        }

        stage('Deploy') {
            agent { label 'deploy-node' }
            steps {
                sh '''
                docker login -u $DOCKERHUB_USER -p $DOCKERHUB_PASS

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
