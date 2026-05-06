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
    command: ["cat"]
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
        stage('Build and Push') {
            steps {
                container('docker') {
                    sh "sleep 15"
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                        sh """
                            echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                            docker build -t ${DOCKER_HUB_USER}/anime-frontend:latest ./frontend
                            docker push ${DOCKER_HUB_USER}/anime-frontend:latest
                            docker build -t ${DOCKER_HUB_USER}/anime-backend:latest .
                            docker push ${DOCKER_HUB_USER}/anime-backend:latest
                        """
                    }
                }
            }
        }
        stage('Deploy to AKS') {
            steps {
                container('kubectl') {
                    // Gunakan variabel KUBECONFIG_CONTENT sebagai 'Secret Text' (bukan file)
                    // Jika kamu masih pakai 'Secret File', pastikan path-nya benar
                    withCredentials([file(credentialsId: "${AKS_CREDENTIALS_ID}", variable: 'KUBECONFIG_PATH')]) {
                        script {
                            sh """
                                # Copy file ke lokasi yang pasti bisa dibaca
                                cp ${KUBECONFIG_PATH} /tmp/kubeconfig
                                export KUBECONFIG=/tmp/kubeconfig
                                
                                # Tambahkan timeout agar tidak menggantung tanpa info
                                kubectl apply -f k8s-manifest.yaml --request-timeout=1m
                                kubectl get pods
                            """
                        }
                    }
                }
            }
        }
    }
    post {
        success {
            echo 'Deployment Berhasil!'
        }
        failure {
            echo 'Build gagal, cek log di atas.'
        }
    }
}