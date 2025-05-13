pipeline {
    agent any
    environment {
        DOCKER_HUB_CREDENTIALS = credentials('dockerhub')
    }
    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/snehabopche/node-ci-cd-k8s.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t snehakurve7/node-ci-cd-k8s .'
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker push snehakurve7/node-ci-cd-k8s'
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

