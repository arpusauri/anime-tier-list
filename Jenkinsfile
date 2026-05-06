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
    volumeMounts:
    - name: dind-storage
      mountPath: /var/lib/docker
    - name: docker-graph-storage
      mountPath: /var/run
  - name: kubectl
    image: bitnami/kubectl:latest
    tty: true
    command: ["cat"]
  volumes:
  - name: dind-storage
    emptyDir: {}
  - name: docker-graph-storage
    emptyDir: {}
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
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                        sh """
                            # Jalankan Docker Daemon di background jika belum jalan
                            dockerd-entrypoint.sh & 
                            sleep 10 
                            
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