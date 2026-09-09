pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
        DOCKERHUB_USER = 'abdulmajeed07'
        IMAGE_NAME = "${DOCKERHUB_USER}/gcxi-demo-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        KUBECONFIG = '/var/jenkins_home/.kube/config'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Prepare Files') {
            steps {
                sh """
                    sed -i 's/BUILD_NUMBER_PLACEHOLDER/${IMAGE_TAG}/' index.html
                    sed -i 's|IMAGE_PLACEHOLDER|${IMAGE_NAME}:${IMAGE_TAG}|' k8s/deployment.yaml
                """
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                sh """
                    echo "\$DOCKERHUB_CREDENTIALS_PSW" | docker login -u "\$DOCKERHUB_CREDENTIALS_USR" --password-stdin
                    docker push ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                    kubectl apply -f k8s/ingress.yaml
                """
            }
        }

        stage('Verify Rollout') {
            steps {
                sh """
                    kubectl rollout status deployment/demo-app -n default --timeout=120s
                    kubectl get pods -n default -l app=demo-app
                """
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful! Build #${IMAGE_TAG} is live."
        }
        failure {
            echo "❌ Pipeline failed — check the logs above."
        }
        always {
            sh "docker logout || true"
        }
    }
}
