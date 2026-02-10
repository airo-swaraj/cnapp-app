pipeline {
    agent any

    environment {
        DOCKERHUB = credentials('dockerhub-creds')
        IMAGE_NAME = "airoswaraj/notes-app"
        KUBECONFIG = "${WORKSPACE}/kubeconfig"
        AWS_REGION = "ap-south-1"
        CLUSTER_NAME = "cnapp-cluster"
        LACEWORK_ACCOUNT = "719551"
    }

    stages {

        stage('Docker Login') {
            steps {
                sh '''
                set -e
                echo $DOCKERHUB_PSW | docker login -u $DOCKERHUB_USR --password-stdin
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                set -e
                docker build -t $IMAGE_NAME:${BUILD_NUMBER} .
                docker tag $IMAGE_NAME:${BUILD_NUMBER} $IMAGE_NAME:latest
                '''
            }
        }

        // ⭐ Scan BEFORE push & deploy
        stage('Scan Image with Lacework') {
            steps {
                withCredentials([
                    string(credentialsId: 'LACEWORK-ACCESS-KEY', variable: 'LW_ACCESS'),
                    string(credentialsId: 'LACEWORK-SECRET-KEY', variable: 'LW_SECRET')
                ]) {
                    sh '''
                    sleep 30

                    lacework vulnerability container scan \
                        index.docker.io \
                        $IMAGE_NAME \
                        ${BUILD_NUMBER} \
                        -a $LACEWORK_ACCOUNT -k $LW_ACCESS -s $LW_SECRET
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                set -e
                docker push $IMAGE_NAME:${BUILD_NUMBER}
                docker push $IMAGE_NAME:latest
                '''
            }
        }

        stage('Configure Kubeconfig') {
            steps {
                sh '''
                set -e
                aws eks update-kubeconfig \
                  --region $AWS_REGION \
                  --name $CLUSTER_NAME \
                  --kubeconfig $KUBECONFIG
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                set -e
                kubectl set image deployment/notes-app \
                notes-app=$IMAGE_NAME:${BUILD_NUMBER}

                kubectl rollout status deployment/notes-app
                '''
            }
        }
    }

    post {
        always {
            sh 'rm -f $KUBECONFIG'
        }
    }
}
