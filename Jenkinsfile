pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'localhost:5000' // or your registry
        KUBECONFIG = credentials('kubeconfig')
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/aayanshahirwar/my-static-website.git'
            }
        }
        
        stage('Build React App') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("react-website:${env.BUILD_ID}")
                }
            }
        }
        
        stage('Push to Registry') {
            steps {
                script {
                    docker.withRegistry("http://${env.DOCKER_REGISTRY}") {
                        docker.image("react-website:${env.BUILD_ID}").push()
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Update deployment with new image
                    sh """
                        kubectl set image deployment/react-website \
                        react-website=${env.DOCKER_REGISTRY}/react-website:${env.BUILD_ID} \
                        --namespace=default
                    """
                    
                    // Wait for rollout to complete
                    sh 'kubectl rollout status deployment/react-website --namespace=default'
                }
            }
        }
        
        stage('Cleanup') {
            steps {
                sh 'docker rmi react-website:${env.BUILD_ID} || true'
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        failure {
            echo 'Pipeline failed!'
        }
        success {
            echo 'Pipeline completed successfully!'
        }
    }
}
