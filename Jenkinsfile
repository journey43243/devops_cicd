pipeline {
    agent any

    environment {
        REGISTRY = "ghcr.io/journey43243/devops_cicd"   // поменяй на свой
        FRONTEND_IMAGE = "${REGISTRY}/frontend"
        BACKEND_IMAGE = "${REGISTRY}/backend"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Frontend Docker Image') {
            steps {
                dir('frontend') {
                    sh "docker build -t ${FRONTEND_IMAGE}:${env.BUILD_NUMBER} ."
                }
            }
        }

        stage('Build Backend Docker Image') {
            steps {
                dir('backend') {
                    sh "docker build -t ${BACKEND_IMAGE}:${env.BUILD_NUMBER} ."
                }
            }
        }

        stage('Login to GHCR') {
            steps {
                withCredentials([string(credentialsId: 'ghcr-token', variable: 'TOKEN')]) {
                    sh """
                        echo $TOKEN | docker login ghcr.io -u journey43243 --password-stdin
                    """
                }
            }
        }

        stage('Push Images') {
            steps {
                sh "docker push ${FRONTEND_IMAGE}:${env.BUILD_NUMBER}"
                sh "docker push ${BACKEND_IMAGE}:${env.BUILD_NUMBER}"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                    kubectl --kubeconfig=/var/jenkins_home/.kube/ci-config get deployment frontend \
                    || kubectl --kubeconfig=/var/jenkins_home/.kube/ci-config create deployment frontend --image=${FRONTEND_IMAGE}:${env.BUILD_NUMBER}
                    kubectl --kubeconfig=/var/jenkins_home/.kube/ci-config get service frontend \
                    || kubectl --kubeconfig=/var/jenkins_home/.kube/ci-config expose deployment frontend --port=80 --type=NodePort
                    kubectl --kubeconfig=/var/jenkins_home/.kube/ci-config set image deployment/frontend \
                    frontend=${FRONTEND_IMAGE}:${env.BUILD_NUMBER}

                    kubectl --kubeconfig=/var/jenkins_home/.kube/ci-config get deployment backend \
                    || kubectl --kubeconfig=/var/jenkins_home/.kube/ci-config create deployment backend --image=${BACKEND_IMAGE}:${env.BUILD_NUMBER}

                    kubectl --kubeconfig=/var/jenkins_home/.kube/ci-config get service backend \
                    || kubectl --kubeconfig=/var/jenkins_home/.kube/ci-config expose deployment backend --port=8080 --type=NodePort

                    kubectl --kubeconfig=/var/jenkins_home/.kube/ci-config set image deployment/backend \
                    backend=${BACKEND_IMAGE}:${env.BUILD_NUMBER}
                """
            }
        }

        stage('Verify Rollout') {
            steps {
                sh "kubectl --kubeconfig=/var/jenkins_home/.kube/ci-config rollout status deployment/frontend"
                sh "kubectl --kubeconfig=/var/jenkins_home/.kube/ci-config rollout status deployment/backend"
            }
        }
    }
}
