pipeline {
    agent any
    
    environment {
        DOCKER_USER = "arpusauri" 
        IMAGE_FRONTEND = "${DOCKER_USER}/anime-frontend:latest"
        IMAGE_BACKEND = "${DOCKER_USER}/anime-backend:latest"
    }
    
    stages {
        stage('1. Checkout Code') {
            steps {
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
                    
                    # Push Image
                    docker push $IMAGE_FRONTEND
                    docker push $IMAGE_BACKEND
                    '''
                }
            }
        }
        
        stage('3. Deploy to AKS') {
            steps {
                // Kita gunakan nama variabel KUBECONFIG_FILE agar tidak bentrok dengan sistem
                withCredentials([file(credentialsId: 'aks-kubeconfig', variable: 'KUBECONFIG_FILE')]) {
                    sh '''
                    # Set environment variable KUBECONFIG ke path file secret dari Jenkins
                    export KUBECONFIG=$KUBECONFIG_FILE
                    
                    # Terapkan konfigurasi Kubernetes
                    kubectl apply -f k8s-manifest.yaml
                    
                    # Restart deployment agar image :latest ditarik ulang
                    kubectl rollout restart deployment anime-frontend
                    kubectl rollout restart deployment anime-backend
                    '''
                }
            }
        }
    }
}
