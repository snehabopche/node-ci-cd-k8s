pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/snehabopche/node-ci-cd-k8s.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def image = "sneha1992/node-ci-cd-k8s:${env.BUILD_NUMBER}"
                    sh "docker build -t ${image} ."
                    sh "docker tag ${image} sneha1992/node-ci-cd-k8s:latest"
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                    script {
                        def image = "sneha1992/node-ci-cd-k8s:${env.BUILD_NUMBER}"
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                        sh "docker push ${image}"
                        sh "docker push sneha1992/node-ci-cd-k8s:latest"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    echo "Deploying to Kubernetes cluster..."
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                    kubectl get svc
                '''
            }
        }
    }
}

