pipeline {

    agent any

    environment {
        IMAGE_NAME = "nginx-demo"
        REGISTRY   = "localhost:8088"
        REPOSITORY = "library"
    }

    stages {

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t ${IMAGE_NAME}:latest .
                """
            }
        }

        stage('Test Image') {
            steps {
                sh """
                docker rm -f nginx-test || true
                docker run -d --name nginx-test -p 8090:80 ${IMAGE_NAME}:latest
                sleep 5
                docker ps
                docker rm -f nginx-test
                """
            }
        }

        stage('Login to Harbor') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'harbor-creds',
                        usernameVariable: 'HARBOR_USER',
                        passwordVariable: 'HARBOR_PASS'
                    )
                ]) {
                    sh """
                    echo "\$HARBOR_PASS" | docker login ${REGISTRY} \
                    --username "\$HARBOR_USER" \
                    --password-stdin
                    """
                }
            }
        }
    }

    post {

        success {
            echo "Pipeline completed successfully."
        }

        failure {
            echo "Pipeline failed."
        }

        always {
            sh 'docker logout localhost:8088 || true'
        }
    }
}
