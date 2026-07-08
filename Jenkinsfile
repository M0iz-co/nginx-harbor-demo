pipeline {

    agent any

    environment {
        IMAGE_NAME = "nginx-demo"
        REGISTRY = "localhost:8088"
        REPOSITORY = "library"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${IMAGE_NAME}:latest .
                """
            }
        }

        stage('Test Docker Image') {
            steps {
                sh """
                    docker rm -f nginx-test || true
                    docker run -d --name nginx-test -p 8090:80 ${IMAGE_NAME}:latest

                    sleep 5

                    curl http://localhost:8090

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

        stage('Tag Image') {
            steps {
                sh """
                    docker tag ${IMAGE_NAME}:latest ${REGISTRY}/${REPOSITORY}/${IMAGE_NAME}:${IMAGE_TAG}
                    docker tag ${IMAGE_NAME}:latest ${REGISTRY}/${REPOSITORY}/${IMAGE_NAME}:latest
                """
            }
        }

        stage('Push Image to Harbor') {
            steps {
                sh """
                    docker push ${REGISTRY}/${REPOSITORY}/${IMAGE_NAME}:${IMAGE_TAG}
                    docker push ${REGISTRY}/${REPOSITORY}/${IMAGE_NAME}:latest
                """
            }
        }

    }

    post {

        success {
            echo "✅ Image successfully pushed to Harbor."
        }

        failure {
            echo "❌ Pipeline failed."
        }

        always {
            sh """
                docker logout ${REGISTRY} || true
                docker rm -f nginx-test || true
            """
        }
    }
}
