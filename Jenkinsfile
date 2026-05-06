pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: build-tools
    image: docker:24.0.5-dind
    securityContext:
      privileged: true
    env:
    - name: DOCKER_TLS_CERTDIR
      value: ""
    tty: true
'''
        }
    }
    environment {
        DOCKER_HUB_USER = 'arpusauri'
        DOCKER_CREDENTIALS_ID = 'docker-hub-credentials'
        AKS_CREDENTIALS_ID = 'aks-kubeconfig'
    }
    stages {
        stage('1. Checkout & Setup') {
            steps {
                container('build-tools') {
                    checkout scm
                    // Install kubectl secara instan di dalam container build-tools
                    sh """
                        curl -LO "https://dl.k8s.io/release/\$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
                        chmod +x kubectl
                        mv kubectl /usr/local/bin/
                    """
                }
            }
        }
        stage('2. Build & Push Docker Images') {
            steps {
                container('build-tools') {
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                        sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"
                        sh "docker build -t ${DOCKER_HUB_USER}/anime-frontend:latest ./frontend"
                        sh "docker push ${DOCKER_HUB_USER}/anime-frontend:latest"
                        sh "docker build -t ${DOCKER_HUB_USER}/anime-backend:latest ."
                        sh "docker push ${DOCKER_HUB_USER}/anime-backend:latest"
                    }
                }
            }
        }
        stage('3. Deploy to AKS') {
            steps {
                container('build-tools') {
                    withCredentials([file(credentialsId: "${AKS_CREDENTIALS_ID}", variable: 'KUBECONFIG')]) {
                        sh "kubectl apply -f k8s-manifest.yaml --kubeconfig=\$KUBECONFIG"
                        sh "kubectl get pods --kubeconfig=\$KUBECONFIG"
                    }
                }
            }
        }
    }
    post {
        success {
            echo 'AKHIRNYA! Deployment Berhasil.'
        }
        failure {
            echo 'Masih gagal? Cek koneksi ke cluster AKS.'
        }
    }
}
