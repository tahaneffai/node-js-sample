pipeline {
    agent any

    environment {
        IMAGE = "tahaneffai/nodeapp:latest"
    }

    stages {
        stage('Clone') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                    sh '''
                        echo "$PASS" | docker login -u "$USER" --password-stdin
                        docker push $IMAGE
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-k3s', variable: 'KUBECONFIG')]) {
                    sh '''
                        set -e
                        kubectl get nodes
                        kubectl apply -f k8s/
                        kubectl set image deployment/nodeapp nodeapp=$IMAGE
                        kubectl rollout status deployment/nodeapp --timeout=180s
                    '''
                }
            }
        }
    }
}
////hhhhhhhhh