pipeline {

    agent any

    environment {

        IMAGE_NAME = "notes-app:latest"
        CONTAINER_NAME = "notes-app-container"
        PORT = "9092"

    }

    stages {

        stage('Checkout') {

            steps {

                git branch: 'master', url: 'https://github.com/santosh662/notes-app.git'

            }
        }

        stage('Build Docker Image') {

            steps {

                echo "======= Building Docker Image ========"

                sh 'docker build -t $IMAGE_NAME .'

            }
        }

        stage('Stop Old Container') {

            steps {

                echo "======= Stopping Old Container ========"

                sh 'docker stop $CONTAINER_NAME || true'

                sh 'docker rm $CONTAINER_NAME || true'

            }
        }

        stage('Run Container') {

            steps {

                echo "======= Running New Container ========"

                sh 'docker run -d --name $CONTAINER_NAME -p $PORT:80 $IMAGE_NAME'

            }
        }

        stage('Verify') {

            steps {

                echo "======= Checking App Response ========"

                sh 'sleep 5'

                sh 'curl -I http://3.110.77.220:$PORT'

            }
        }
    }

    post {

        success {

            echo "Notes app deployed successfully"

            echo "Application URL: http://3.110.77.220:$PORT"

        }

        failure {

            echo "Notes app deployment failed. Check logs."

        }
    }
}
