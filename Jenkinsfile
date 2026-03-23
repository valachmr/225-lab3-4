pipeline {
    agent any 

    environment {
        DOCKER_CREDENTIALS_ID = 'roseaw-dockerhub'  
        DOCKER_IMAGE = 'cithit/valachmr'                          //<-----change this to your MiamiID!
        IMAGE_TAG = "build-${BUILD_NUMBER}"
        GITHUB_URL = 'https://github.com/valachmr/225-lab3-4.git' //<-----change this to match this new repository!
        KUBECONFIG = credentials('valachmr-225-sp26')             //<-----change this to match your kubernetes credentials (MiamiID-225)!  1 More change on line 63!
    }

     stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']],
                          userRemoteConfigs: [[url: "${GITHUB_URL}"]]])
            }
        }

        stage('Lint HTML') {
            steps {
                sh 'npm install htmlhint --save-dev'
                sh 'npx htmlhint *.html'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${IMAGE_TAG}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDENTIALS_ID}") {
                        docker.image("${DOCKER_IMAGE}:${IMAGE_TAG}").push()
                    }
                }
            }
        }

        stage('Deploy to Dev Environment') {
            steps {
                steps {
        withCredentials([file(credentialsId: 'valachmr-225-sp26', variable: 'KUBECONFIG')]) {
            sh '''
                export KUBECONFIG=$KUBECONFIG

                echo "=== DEBUG: kubeconfig ==="
                kubectl config view

                echo "=== DEBUG: cluster access ==="
                kubectl get pods

                sed -i "s|${DOCKER_IMAGE}:latest|${DOCKER_IMAGE}:${IMAGE_TAG}|" deployment-dev.yaml

                kubectl apply -f deployment-dev.yaml
            '''
        }
    }
            }
        }
        stage('Deploy to Prod Environment') {
            steps {
                withCredentials([file(credentialsId: 'valachmr-225-sp26', variable: 'KUBECONFIG')]) {
                sh '''
                    export KUBECONFIG=$KUBECONFIG

                    sed -i "s|${DOCKER_IMAGE}:latest|${DOCKER_IMAGE}:${IMAGE_TAG}|" deployment-prod.yaml

                    kubectl apply -f deployment-prod.yaml
                '''
                }
            }
        }
        stage('Check Kubernetes Cluster') {
            steps {
                script {
                    sh "kubectl get pods"
                    sh "kubectl get services"
                    sh "kubectl get deploy"
                }
            }
        }
    }

    post {

        success {
            slackSend color: "good", message: "Build Completed: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
        }
            
        unstable {
            slackSend color: "warning", message: "Build Completed: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
        }
            
        failure {
            slackSend color: "danger", message: "Build Completed: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
        }
    }
}
