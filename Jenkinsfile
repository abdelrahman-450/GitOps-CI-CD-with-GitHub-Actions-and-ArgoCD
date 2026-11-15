
pipeline {
    agent any

    environment {
        IMAGE_REPO = '3booda24/jenkinsfreestyle'
        IMAGE_TAG = 'latest'
    }

    stages {

        stage('Checkout Source Code') {
            steps {
                git branch: 'Master',
                    url: 'https://github.com/abdelrahmanonline4/GitOps-CI-CD-with-GitHub-Actions-and-ArgoCD.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t ${IMAGE_REPO}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Test Docker Container') {
            steps {
                sh '''
                    docker rm -f testcontainer || true

                    docker run -d --name testcontainer -p 3000:3000 ${IMAGE_REPO}:${IMAGE_TAG}

                    sleep 10
                    curl --fail http://localhost:3000
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-cred',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_TOKEN'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_TOKEN" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${IMAGE_REPO}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Production Approval Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    input(
                        message: 'Approve Deployment to GKE?',
                        ok: 'Approve Deployment'
                    )
                }
            }
        }

        stage('Deploy to GKE') {
            steps {
                sh 'kubectl apply -f kubernetes/'
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    kubectl get deployments
                    kubectl get pods
                    kubectl get services
                '''
            }
        }
    }

    post {
        always {
            sh 'docker rm -f testcontainer || true'
        }

        success {
            echo 'Pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed.'
        }
    }
}
