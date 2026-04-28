pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }

        stage('Validate Kubernetes Manifests') {
            steps {
                echo 'Validating Kubernetes YAML files...'
                sh '''
                    if ! command -v kubeconform >/dev/null 2>&1; then
                      echo "Installing kubeconform..."
                      curl -L https://github.com/yannh/kubeconform/releases/latest/download/kubeconform-linux-amd64.tar.gz -o kubeconform.tar.gz
                      tar -xzf kubeconform.tar.gz
                      sudo mv kubeconform /usr/local/bin/
                    fi

                    kubeconform -summary -ignore-missing-schemas kubernetes-manifests
                '''
            }
        }

        stage('Trivy Security Scan') {
            steps {
                echo 'Running Trivy vulnerability scan...'
                sh '''
                    trivy fs --severity HIGH,CRITICAL . || true
                '''
            }
        }

        stage('Archive Trivy Report') {
            steps {
                echo 'Saving Trivy scan report...'
                sh '''
                    trivy fs --severity HIGH,CRITICAL . > trivy-jenkins-report.txt || true
                '''
                archiveArtifacts artifacts: 'trivy-jenkins-report.txt', fingerprint: true
            }
        }
    }

    post {
        success {
            echo 'Jenkins pipeline completed successfully.'
        }

        failure {
            echo 'Jenkins pipeline failed. Check the console output.'
        }
    }
}