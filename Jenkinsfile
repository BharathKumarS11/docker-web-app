pipeline {
    agent any

    environment {
        IMAGE_NAME = 'bharathkumar11/devops-web-app'
        IMAGE_TAG  = "${BUILD_NUMBER}"
    }

    stages {

        stage('Build') {
            steps {
                echo 'Building Docker image...'

                sh '''
                    docker build \
                        -t $IMAGE_NAME:$IMAGE_TAG \
                        .
                '''
            }
        }

        stage('Test') {
            steps {
                echo 'Testing Docker container...'

                sh '''
                    docker run -d \
                        --name jenkins-test-app \
                        -p 8080:80 \
                        $IMAGE_NAME:$IMAGE_TAG

                    sleep 3

                    curl --fail http://localhost:8080
                '''
            }
        }

        stage('Push') {
            steps {
                echo 'Pushing Docker image...'

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | \
                        docker login \
                        --username "$DOCKER_USERNAME" \
                        --password-stdin

                        docker push $IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }
    }

    post {
        always {
            sh 'docker rm -f jenkins-test-app || true'
        }

        success {
            echo 'CI/CD pipeline completed successfully!'
        }

        failure {
            echo 'CI/CD pipeline failed. Check the logs.'
        }
    }
}