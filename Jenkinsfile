pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: docker
    image: docker:24.0.5-dind
    securityContext:
      privileged: true
    env:
    - name: DOCKER_TLS_CERTDIR
      value: ""
  - name: kubectl
    image: bitnami/kubectl:latest
    tty: true
    command: ["sleep"]
    args: ["infinity"]
'''
        }
    }
    environment {
        DOCKER_HUB_USER = 'arpusauri'
        DOCKER_CREDENTIALS_ID = 'docker-hub-credentials'
        AKS_CREDENTIALS_ID = 'aks-kubeconfig'
    }
    stages {
        stage('Build & Push') {
            steps {
                container('docker') {
                    sh "sleep 15"
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
        stage('Deploy') {
            steps {
                // Kita gunakan container default (jnlp) saja agar lebih stabil
                container('jnlp') { 
                    withCredentials([file(credentialsId: "${AKS_CREDENTIALS_ID}", variable: 'KUBECONFIG')]) {
                        script {
                            // Kita install kubectl secara instan di dalam agent
                            sh """
                                curl -LO "https://dl.k8s.io/release/\$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
                                chmod +x kubectl
                                ./kubectl apply -f k8s-manifest.yaml --kubeconfig=${KUBECONFIG}
                                ./kubectl get pods --kubeconfig=${KUBECONFIG}
                            """
                        }
                    }
                }
            }
        }
    post {
        success {
            echo 'BERHASIL TOTAL! Cek External IP di Azure sekarang.'
        }
    }
}