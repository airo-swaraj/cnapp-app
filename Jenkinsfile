pipeline {
    agent any

    environment {
        DOCKERHUB = credentials('dockerhub-creds')
        IMAGE_NAME = "airo-swaraj/notes-app"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-creds',
                    url: 'https://github.com/airo-swaraj/cnapp-app.git'
            }
        }

        stage('Docker Login') {
            steps {
                sh '''
                echo $DOCKERHUB_PSW | docker login -u $DOCKERHUB_USR --password-stdin
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                cd app-code
                docker build -t $IMAGE_NAME:latest .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                docker push $IMAGE_NAME:latest
                '''
            }
        }
    }
}
