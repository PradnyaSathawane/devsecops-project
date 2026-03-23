pipeline {
    agent any

    environment {
     NVD_API_KEY = '20680eee-11c2-4ae5-a483-9dc286e45b1e'
 }

    stages {

        stage('Build') {
            steps {
                sh 'docker build -t devsecops-app .'
            }
        }

        stage('SAST Scan') {
            steps {
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                sh '/opt/sonar-scanner/bin/sonar-scanner -Dsonar.projectKey=devsecops -Dsonar.sources=. -Dsonar.host.url=http://127.0.0.1:9000 -Dsonar.login=$SONAR_TOKEN' 
	    	}
            }
        }

        stage('Dependency Scan') {
            steps {
		sh '''
                ./dependency-check/bin/dependency-check.sh \
                --project devsecops-project \
                --scan . \
                --format HTML \
                --out dependency-check-report \
                --nvdApiKey $NVD_API_KEY || true
                '''
            }
        }

        stage('Image Scan') {
            steps {
                sh '''
                export TRIVY_CACHE_DIR=/tmp/trivy-cache
                export TRIVY_TIMEOUT=10m

                mkdir -p $TRIVY_CACHE_DIR

                trivy image \
                --cache-dir $TRIVY_CACHE_DIR \
                --timeout 10m \
                --severity HIGH,CRITICAL \
                --exit-code 0 \
                devsecops-app
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                export KUBECONFIG=/var/lib/jenkins/.kube/config
                kubectl config use-context kind-devsecops-cluster
                kubectl apply -f k8s/
                '''

            }
        }

    }
}
