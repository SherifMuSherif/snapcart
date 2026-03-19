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
                sh """
            # Download kubectl
            curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
            chmod +x kubectl
            
            kubectl apply -f ${K8S_DIR}/namespace.yaml
            kubectl apply -f ${K8S_DIR}/deployment.yaml
            kubectl apply -f ${K8S_DIR}/service.yaml

            kubectl rollout status deployment/snapcart-deployment \
                -n ${NAMESPACE} --timeout=120s
        """
            }
        }
    }

   
}
