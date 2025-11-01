pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'aliwazeer'
        DOCKERHUB_CREDENTIALS = credentials('dockerhub') // Jenkins credential ID
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
                    # إعداد ملف config.json لتسجيل الدخول على DockerHub
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

                    # تنفيذ Kaniko لبناء ودفع الصورة
                    /kaniko/executor \
                      --context ./backend \
                      --dockerfile ./backend/Dockerfile \
                      --destination $DOCKERHUB_USER/backend:$IMAGE_TAG \
                      --cleanup
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo '☸️ Deploying to Kubernetes...'
                sh '''
                    # إنشاء namespace لو مش موجود
                    kubectl get ns $K8S_NAMESPACE || kubectl create ns $K8S_NAMESPACE

                    # تطبيق ملفات الـ Kubernetes
                    kubectl apply -f K8S/ -n $K8S_NAMESPACE

                    # تحديث صورة backend
                    kubectl set image deployment/backend-deployment backend=$DOCKERHUB_USER/backend:$IMAGE_TAG -n $K8S_NAMESPACE

                    # الانتظار حتى ينتهي rollout
                    kubectl rollout status deployment/backend-deployment -n $K8S_NAMESPACE
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
