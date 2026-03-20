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
		sh '''
                ./dependency-check/bin/dependency-check.sh \
                --project "devsecops-project" \
                --scan . \
                --format HTML \
                --out dependency-check-report \
                --nvdApiDelay 2000 \
                --disableCentral
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
