pipeline {
    agent any
    environment {
        TRIVY_PATH = '/usr/local/bin/trivy'  // Path to Trivy binary
    }
    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    def image = "sneha1992/node-ci-cd-k8s:${env.BUILD_NUMBER}"
                    // Build Docker image and tag it
                    sh "docker build -t sneha1992/node-ci-cd-k8s:${env.BUILD_NUMBER} ."
                    sh "docker tag sneha1992/node-ci-cd-k8s:${env.BUILD_NUMBER} sneha1992/node-ci-cd-k8s:latest"
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                script {
                    // Run Trivy scan for vulnerabilities on the image
                    sh "${TRIVY_PATH} image --severity HIGH,CRITICAL --no-progress --exit-code 1 sneha1992/node-ci-cd-k8s:${env.BUILD_NUMBER}"
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                    script {
                        // Log in to DockerHub and push the image
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                        sh "docker push sneha1992/node-ci-cd-k8s:${env.BUILD_NUMBER}"
                        sh "docker push sneha1992/node-ci-cd-k8s:latest"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-eks-credentials'
                ]]) {
                    script {
                        // Set up AWS CLI and kubeconfig for EKS
                        sh """
                            echo "Configuring AWS CLI and kubeconfig..."
                            aws eks update-kubeconfig --region us-east-1 --name jenkins-eks

                            echo "Deploying to Kubernetes cluster..."
                            kubectl apply --validate=false -f k8s/deployment.yaml
                            kubectl apply --validate=false -f k8s/service.yaml
                            kubectl get svc
                        """
                    }
                }
            }
        }
    }
}

