pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'aliwazeer'
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
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

        stage('Build & Push Image using Kaniko') {
            steps {
                echo '🚀 Building and pushing image with Kaniko...'
                sh '''
                    mkdir -p /tmp/.docker

                    cat <<EOF > /tmp/.docker/config.json
                    {
                        "auths": {
                            "https://index.docker.io/v1/": {
                                "auth": "$(echo -n "$DOCKERHUB_USER:$DOCKERHUB_CREDENTIALS_PSW" | base64)"
                            }
                        }
                    }
                    EOF

                    /kaniko/executor \
                      --context `pwd` \
                      --dockerfile Dockerfile \
                      --destination $DOCKERHUB_USER/backend:$IMAGE_TAG \
                      --cleanup
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo '☸️ Deploying to Kubernetes...'
                sh '''
                    kubectl apply -f K8S/ -n $K8S_NAMESPACE
                    kubectl set image deployment/backend-deployment backend=aliwazeer/backend:${BUILD_NUMBER} -n dev

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
