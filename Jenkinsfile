pipeline {
    agent any

    environment {
        AWS_REGION     = 'us-east-2'
        AWS_ACCOUNT_ID = '220912169334'
        ECR_REPOSITORY = 'devops-tech-challenge-2-app'
        EKS_CLUSTER    = 'tech-challenge-2-eks'
        HELM_RELEASE   = 'tech-challenge-2'
        CHART_PATH     = 'helm/tech-challenge-2'

        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        IMAGE_TAG    = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build \
                      -t ${ECR_REPOSITORY}:${IMAGE_TAG} \
                      ./app
                '''
            }
        }

        stage('Push to ECR') {
            steps {
                sh '''
                    aws ecr get-login-password --region ${AWS_REGION} | \
                    docker login --username AWS --password-stdin ${ECR_REGISTRY}

                    docker tag \
                      ${ECR_REPOSITORY}:${IMAGE_TAG} \
                      ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}

                    docker push \
                      ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}
                '''
            }
        }

        stage('Configure EKS') {
            steps {
                sh '''
                    aws eks update-kubeconfig \
                      --region ${AWS_REGION} \
                      --name ${EKS_CLUSTER}
                '''
            }
        }

        stage('Deploy with Helm') {
            steps {
                sh '''
                    helm upgrade --install ${HELM_RELEASE} ${CHART_PATH} \
                      --set image.repository=${ECR_REGISTRY}/${ECR_REPOSITORY} \
                      --set image.tag=${IMAGE_TAG}

                    kubectl rollout status \
                      deployment/${HELM_RELEASE} \
                      --timeout=180s
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    kubectl get deployment
                    kubectl get pods
                    kubectl get hpa
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD deployment completed successfully.'
        }

        failure {
            echo 'CI/CD pipeline failed. Review the failed stage.'
        }
    }
}