pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'YOUR_DOCKERHUB_USERNAME/ticket-reservation'
        TOMCAT_URL = 'http://tomcat-host:8080'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Test') {
            steps {
                sh 'mvn -B clean verify'
            }
        }

        stage('Deploy WAR to Tomcat') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'tomcat-manager',
                    usernameVariable: 'TOMCAT_USER',
                    passwordVariable: 'TOMCAT_PASSWORD'
                )]) {
                    sh '''
                        curl --fail --silent --show-error \
                          --user "$TOMCAT_USER:$TOMCAT_PASSWORD" \
                          --upload-file target/ticket-reservation.war \
                          "$TOMCAT_URL/manager/text/deploy?path=/ticket-reservation&update=true"
                    '''
                }
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKERHUB_USER',
                    passwordVariable: 'DOCKERHUB_TOKEN'
                )]) {
                    sh '''
                        set +x
                        echo "$DOCKERHUB_TOKEN" | docker login --username "$DOCKERHUB_USER" --password-stdin
                        docker build -t "$DOCKER_IMAGE:$BUILD_NUMBER" .
                        docker tag "$DOCKER_IMAGE:$BUILD_NUMBER" "$DOCKER_IMAGE:latest"
                        docker push "$DOCKER_IMAGE:$BUILD_NUMBER"
                        docker push "$DOCKER_IMAGE:latest"
                        docker logout
                    '''
                }
            }
        }
    }

    post {
        success {
            emailext(
                to: 'YOUR_EMAIL',
                subject: "SUCCESS: ${JOB_NAME} #${BUILD_NUMBER}",
                body: "Build succeeded: ${BUILD_URL}"
            )
        }
        failure {
            emailext(
                to: 'YOUR_EMAIL',
                subject: "FAILED: ${JOB_NAME} #${BUILD_NUMBER}",
                body: "Build failed. Check: ${BUILD_URL}"
            )
        }
    }
}
