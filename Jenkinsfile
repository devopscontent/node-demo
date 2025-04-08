pipeline {
    agent any

    environment {
        IMAGE_NAME = 'samintelli/node-app'
        IMAGE_TAG = 'latest' // You can change this to BUILD_NUMBER or Git SHA if needed
    }

    stages {
        stage('Clone') {
            steps {
                git 'https://github.com/devopscontent/node-demo.git'
            }
        }

        stage('Build') {
            steps {
                sh 'npm install'
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                    sh """
                        echo $DOCKERHUB_PASSWORD | docker login -u $DOCKERHUB_USERNAME --password-stdin
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('K8s Deploy') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-cred-id', variable: 'KUBECONFIG')]) {
                    sh """
                        # Apply manifests (if not already applied)
                        kubectl --kubeconfig=$KUBECONFIG apply -f deployment.yaml
                        
                        # Set new image for deployment
                        kubectl --kubeconfig=$KUBECONFIG set image deployment/nodejs-app nodejs-container=${IMAGE_NAME}:${IMAGE_TAG} --record

                        # Optional: Wait for rollout to complete
                        kubectl --kubeconfig=$KUBECONFIG rollout status deployment/nodejs-app
                    """
                }
            }
        }
    }

    post {
        success {
            echo ' Deployed successfully to Kubernetes!'
        }
        failure {
            echo ' Deployment failed!'
        }
        always {
            // Optional cleanup
            sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || true"
        }
    }
}
