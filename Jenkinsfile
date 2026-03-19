pipeline {
    agent any

    environment {
        IMAGE_NAME = 'snapcart'
        IMAGE_TAG  = 'latest'
        NAMESPACE  = 'snapcart'
        K8S_DIR    = 'k8s'
    }

    stages {
        // all stages go here

        stage('Checkout') {
            steps {
                echo 'Pulling latest code from GitHub...'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
            docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
        """
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-file', variable: 'KUBECONFIG')]) {
                    sh '''
                # 1. Download & Prep kubectl
                curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
                chmod +x ./kubectl

                # 2. Fix the KUBECONFIG IP 
                # We replace 127.0.0.1 with host.docker.internal (for Docker Desktop)
                # If you are on pure Linux, replace 'host.docker.internal' with your host IP (e.g., 172.17.0.1)
                sed "s/127.0.0.1/host.docker.internal/g" $TEMP_KUBECONFIG > ./kubeconfig.modified
                
                export KUBECONFIG=./kubeconfig.modified

                # 3. Apply manifests
                ./kubectl apply -f k8s/namespace.yaml --validate=false
                ./kubectl apply -f k8s/deployment.yaml --validate=false
                ./kubectl apply -f k8s/service.yaml --validate=false

                ./kubectl rollout status deployment/snapcart-deployment \
                    -n ${NAMESPACE} --timeout=120s
            '''
                }
            }
        }
    }

}
