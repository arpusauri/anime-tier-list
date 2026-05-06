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
    # HAPUS command ["cat"] di sini agar daemon docker bisa start otomatis
    env:
    - name: DOCKER_TLS_CERTDIR
      value: ""
  - name: kubectl
    image: bitnami/kubectl:latest
    tty: true
    command: ["cat"]
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
                    // Kita beri waktu 15 detik agar daemon docker benar-benar 'bangun'
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
                container('kubectl') {
                    withCredentials([file(credentialsId: "${AKS_CREDENTIALS_ID}", variable: 'KUBECONFIG')]) {
                        sh "kubectl apply -f k8s-manifest.yaml --kubeconfig=\$KUBECONFIG"
                    }
                }
            }
        }
    }
}