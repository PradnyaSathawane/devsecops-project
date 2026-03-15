pipeline {
    agent any

    stages {

        stage('Build') {
            steps {
                sh 'docker build -t devsecops-app .'
            }
        }

        stage('SAST Scan') {
            steps {
                sh 'sonar-scanner'
            }
        }

        stage('Dependency Scan') {
            steps {
                sh './dependency-check/bin/dependency-check.sh --project test --scan .'
            }
        }

        stage('Image Scan') {
            steps {
                sh 'trivy image devsecops-app'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f k8s/'
            }
        }

    }
}
