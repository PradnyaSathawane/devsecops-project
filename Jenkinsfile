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
		timeout(time: 25, unit: 'MINUTES') {
               	    sh '''
                    /opt/dependency-check/bin/dependency-check.sh \
                    --project test \
                    --scan . \
                    --nvdApiKey  20680eee-11c2-4ae5-a483-9dc286e45b1e \
                    --format HTML \
	            --noupdate
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
