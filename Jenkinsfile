pipeline {

    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t nginx-demo .'
            }
        }

        stage('Test Image') {
            steps {
                sh '''
                docker rm -f nginx-test || true
                docker run -d --name nginx-test -p 8090:80 nginx-demo
                docker ps
                docker rm -f nginx-test
                '''
            }
        }

    }

}
