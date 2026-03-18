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
                sh '/opt/sonar-scanner/bin/sonar-scanner -Dsonar.projectKey=devsecops -Dsonar.sources=. -Dsonar.host.url=http://127.0.0.1:9000 -Dsonar.token=squ_ae071c847fa0c8a0b56eef61bdff446db8011abe'
            }
        }

        stage('Dependency Scan') {
            steps {
		echo "Skipping heavy Dependency Check for faster pipeline"
            }
        }

        stage('Image Scan') {
            steps {
                sh '''
        	trivy image --exit-code 0 --severity HIGH,CRITICAL devsecops-app
        	'''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f k8s/'
            }
        }

    }
}
