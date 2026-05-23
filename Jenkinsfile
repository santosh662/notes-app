pipeline {

    agent { label 'dev' }

    environment {

        DOCKER_CREDS = credentials('Docker_hub_id_pwd')
        IMAGE_NAME = "notes-app"
        IMAGE_TAG = "v1.${BUILD_NUMBER}"
        CONTAINER_NAME = "notes_app_container"
        PORT = "9092"

    }

    stages {

        stage('Checkout Code') {

            steps {
                git(
                    branch: 'master',
                    url: 'https://github.com/santosh662/notes-app.git'
                )
            }
        }

        stage('Build Docker Image') {

            steps {
                echo "===== Building Docker Image ====="
                sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
            }
        }

        stage('Tag Docker Image') {

            steps {
                echo "===== Tagging Docker Image ====="
                sh 'docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${DOCKER_CREDS_USR}/${IMAGE_NAME}:${IMAGE_TAG}'
            }
        }

        stage('Docker Login & Push') {

            steps {
                echo "===== Docker Login ====="

                sh '''
                echo "$DOCKER_CREDS_PSW" | docker login -u "$DOCKER_CREDS_USR" --password-stdin
                '''

                echo "===== Pushing Image ====="

                sh '''
                docker push ${DOCKER_CREDS_USR}/${IMAGE_NAME}:${IMAGE_TAG}
                '''

                sh 'docker logout'
            }
        }

        stage('Stop Old Container') {

            steps {
                echo "===== Stopping Old Container ====="
                sh 'docker stop ${CONTAINER_NAME} || true'
                sh 'docker rm ${CONTAINER_NAME} || true'
            }
        }

        stage('Pull Image') {

            steps {
                echo "===== Pulling Image ====="
                sh 'docker pull ${DOCKER_CREDS_USR}/${IMAGE_NAME}:${IMAGE_TAG}'
            }
        }

        stage('Run Container') {

            steps {
                echo "===== Running Container ====="

                sh '''
                docker run -d \
                --name ${CONTAINER_NAME} \
                --restart unless-stopped \
                -p ${PORT}:80 \
                -v notes_data:/data \
                ${DOCKER_CREDS_USR}/${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }

        stage('Verify Deployment') {

            steps {

                script {

                    def status = sh(
                        script: '''
                        for i in {1..10}; do
                            curl -f http://3.110.197.175:${PORT} && exit 0
                            sleep 5
                        done
                        exit 1
                        ''',
                        returnStatus: true
                    )

                    if (status != 0) {
                        error("Application is not reachable")
                    }
                }
            }
        }
    }

    post {

        success {

            echo "===== Deployment Successful ====="
            echo "Application URL: http://13.201.54.88:${PORT}"

            emailext (
                subject: "Build Was Successful",
                body: "Build was successful. Congratulations!",
                to: "patelsantosh441@gmail.com"
            )
        }

        failure {

            echo "===== Deployment Failed ====="

            emailext (
                subject: "Ops! Build Failed",
                body: "Bhai build fail ho gya jaldi idhr dekh to.",
                to: "patelsantosh441@gmail.com"
            )
        }

        always {

            script {
                sh 'docker image prune -f || true'
            }
        }
    }
}
