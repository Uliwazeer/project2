pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'aliwazeer'
        DOCKERHUB_CREDENTIALS = credentials('dockerhub') // Jenkins credential ID
        IMAGE_TAG = "${BUILD_NUMBER}"
        K8S_NAMESPACE = 'dev'
        USE_MINIKUBE = false // true لو عايز تبني على Minikube، false لو هتبني على DockerHub
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo '📦 Fetching source code...'
                checkout scm
            }
        }

        stage('Build & Push Image') {
            steps {
                script {
                    if (env.USE_MINIKUBE == 'true') {
                        echo '🐳 Building Docker image inside Minikube...'
                        sh '''
                            eval $(minikube docker-env)
                            docker build -t backend:${IMAGE_TAG} ./backend
                        '''
                    } else {
                        echo '🚀 Building and pushing image to DockerHub using Kaniko...'
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
                              --context ./backend \
                              --dockerfile ./backend/Dockerfile \
                              --destination $DOCKERHUB_USER/backend:$IMAGE_TAG \
                              --cleanup
                        '''
                    }
                }
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
                    if [ "$USE_MINIKUBE" = "true" ]; then
                        kubectl set image deployment/backend-deployment backend=backend:${IMAGE_TAG} -n $K8S_NAMESPACE
                    else
                        kubectl set image deployment/backend-deployment backend=$DOCKERHUB_USER/backend:${IMAGE_TAG} -n $K8S_NAMESPACE
                    fi
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
