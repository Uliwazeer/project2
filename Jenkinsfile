pipeline {
    // نستخدم Agent عليه Docker CLI (label = docker)
    agent { label 'docker' }

    environment {
        DOCKERHUB_USER = 'aliwazeer'
        DOCKERHUB_CREDENTIALS = credentials('dockerhub') // تأكد إن الـ ID ده موجود في Jenkins credentials
        IMAGE_TAG = "${BUILD_NUMBER}"
        K8S_NAMESPACE = 'dev'
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo '📦 Fetching source code...'
                checkout scm
            }
        }

        stage('Docker Login') {
            steps {
                echo '🔑 Logging in to DockerHub...'
                sh '''
                    echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_USER --password-stdin
                '''
            }
        }

        stage('Build Docker Images') {
            steps {
                echo '🐳 Building Docker images...'
                sh '''
                    docker build -t $DOCKERHUB_USER/backend:$IMAGE_TAG ./backend
                    docker build -t $DOCKERHUB_USER/nginx:$IMAGE_TAG ./nginx
                '''
            }
        }

        stage('Push Docker Images') {
            steps {
                echo '🚀 Pushing Docker images to DockerHub...'
                sh '''
                    docker push $DOCKERHUB_USER/backend:$IMAGE_TAG
                    docker push $DOCKERHUB_USER/nginx:$IMAGE_TAG
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo '☸️ Deploying to Kubernetes...'
                sh '''
                    kubectl apply -f K8S/ -n $K8S_NAMESPACE
                    kubectl set image deployment/backend backend=$DOCKERHUB_USER/backend:$IMAGE_TAG -n $K8S_NAMESPACE
                    kubectl set image deployment/proxy proxy=$DOCKERHUB_USER/nginx:$IMAGE_TAG -n $K8S_NAMESPACE
                '''
            }
        }

        stage('Smoke Test') {
            steps {
                echo '🧪 Running health check...'
                sh '''
                    sleep 10
                    kubectl get pods -n $K8S_NAMESPACE
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Deployment Successful!'
        }
        failure {
            echo '❌ Deployment Failed!'
        }
    }
}
