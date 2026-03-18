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
		timeout(time: 10, unit: 'MINUTES') {
               	    sh '''
                    /opt/dependency-check/bin/dependency-check.sh \
                    --project test \
                    --scan . \
                    --nvdApiKey 59875C63-9822-F111-8369-129478FCB64D \
                    --format HTML
       	            '''
	       }
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
