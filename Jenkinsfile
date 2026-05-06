pipeline {
    agent any
    
    environment {
        DOCKER_USER = "arpusauri" // Ganti dengan username Docker Hub
        IMAGE_FRONTEND = "${DOCKER_USER}/anime-frontend:latest"
        IMAGE_BACKEND = "${DOCKER_USER}/anime-backend:latest"
    }
    
    stages {
        stage('1. Checkout Code') {
            steps {
                // Mengambil kode dari GitHub (secara default akan menggunakan repo tempat Jenkinsfile ini berada)
                checkout scm
            }
        }
        
        stage('2. Build & Push Docker Images') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER_CRED')]) {
                    sh '''
                    # Login ke Docker Hub
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER_CRED" --password-stdin
                    
                    # Build Image
                    docker build -t $IMAGE_FRONTEND -f FrontendDockerfile .
                    docker build -t $IMAGE_BACKEND -f BackendDockerfile .
                    
                    # Push Image ke Registry
                    docker push $IMAGE_FRONTEND
                    docker push $IMAGE_BACKEND
                    '''
                }
            }
        }
        
        stage('3. Deploy to AKS') {
            steps {
                withCredentials([file(credentialsId: 'aks-kubeconfig', variable: 'KUBECONFIG')]) {
                    sh '''
                    export KUBECONFIG=$KUBECONFIG
                    
                    # Aplikasikan file YAML ke cluster Kubernetes
                    kubectl apply -f k8s-manifest.yaml
                    
                    # Restart deployment agar selalu pull image terbaru jika menggunakan tag :latest
                    kubectl rollout restart deployment anime-frontend
                    kubectl rollout restart deployment anime-backend
                    '''
                }
            }
        }
    }
}
