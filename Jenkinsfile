pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }

        stage('Install kubeconform') {
            steps {
                echo 'Installing kubeconform locally...'
                sh '''
                    mkdir -p tools
                    curl -L https://github.com/yannh/kubeconform/releases/latest/download/kubeconform-linux-amd64.tar.gz -o kubeconform.tar.gz
                    tar -xzf kubeconform.tar.gz
                    mv kubeconform tools/kubeconform
                    chmod +x tools/kubeconform
                '''
            }
        }

        stage('Validate Kubernetes Manifests') {
            steps {
                echo 'Validating Kubernetes YAML files...'
                sh '''
                    ./tools/kubeconform -summary -ignore-missing-schemas kubernetes-manifests
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
                sh '''
                    trivy fs --severity HIGH,CRITICAL . > trivy-jenkins-report.txt || true
                '''
                archiveArtifacts artifacts: 'trivy-jenkins-report.txt', fingerprint: true
            }
        }
    }
}