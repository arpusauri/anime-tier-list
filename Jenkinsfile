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
    tty: true
    command: ["cat"]
    env:
    - name: DOCKER_TLS_CERTDIR
        value: ""
    - name: kubectl
        image: bitnami/kubectl:latest
        tty: true
        command: ["cat"]
    - name: DOCKER_HOST
        value: "tcp://localhost:2375"
'''
        }
    }
    environment {
        DOCKER_HUB_USER = 'arpusauri'
        DOCKER_CREDENTIALS_ID = 'docker-hub-credentials'
        AKS_CREDENTIALS_ID = 'aks-kubeconfig'
    }
    stages {
        stage('1. Checkout Code') {
            steps {
                // Checkout dilakukan di luar container block agar menggunakan default jnlp agent
                checkout scm
            }
        }
        stage('2. Build & Push Docker Images') {
            steps {
                container('docker') {
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                        sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"
                        sh "docker info"
                        // Build Frontend
                        sh "docker build -t ${DOCKER_HUB_USER}/anime-frontend:latest ./frontend"
                        sh "docker push ${DOCKER_HUB_USER}/anime-frontend:latest"
                        
                        // Build Backend (File ada di root)
                        sh "docker build -t ${DOCKER_HUB_USER}/anime-backend:latest ."
                        sh "docker push ${DOCKER_HUB_USER}/anime-backend:latest"
                    }
                }
            }
        }
        stage('3. Deploy to AKS') {
            steps {
                container('kubectl') {
                    withCredentials([file(credentialsId: "${AKS_CREDENTIALS_ID}", variable: 'KUBECONFIG')]) {
                        // Gunakan --kubeconfig agar kubectl tahu cluster mana yang dituju
                        sh "kubectl apply -f k8s-manifest.yaml --kubeconfig=\$KUBECONFIG"
                        sh "kubectl get pods --kubeconfig=\$KUBECONFIG"
                    }
                }
            }
        }
    }
    post {
        success {
            echo 'YES! Berhasil dideploy ke AKS.'
        }
        failure {
            echo 'Build gagal, periksa kembali log di atas.'
        }
    }
}
