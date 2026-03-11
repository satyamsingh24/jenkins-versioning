pipeline {
    agent { label 'agent-1' }
    
    environment {
        REPO_URL    = 'https://github.com/<username>/jenkins-versioning.git'
        IMAGE_NAME  = '176583374037.dkr.ecr.ap-south-1.amazonaws.com/myapp-repo'
        AWS_REGION  = 'ap-south-1'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git credentialsId: 'github-creds',
                    url: "${REPO_URL}",
                    branch: 'main'
            }
        }
        
        stage('Generate Version') {
            steps {
                script {
                    // Version banao: v1.0.BUILD_NUMBER
                    env.VERSION = "v1.0.${BUILD_NUMBER}"
                    echo "Version: ${env.VERSION}"
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${IMAGE_NAME}:${VERSION} .
                    docker tag ${IMAGE_NAME}:${VERSION} ${IMAGE_NAME}:latest
                """
            }
        }
        
        stage('Push to ECR') {
            steps {
                sh """
                    aws ecr get-login-password --region ${AWS_REGION} | \
                    docker login --username AWS \
                    --password-stdin ${IMAGE_NAME}
                    
                    docker push ${IMAGE_NAME}:${VERSION}
                    docker push ${IMAGE_NAME}:latest
                """
            }
        }
        
        stage('Git Tag') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'github-creds',
                    usernameVariable: 'GIT_USER',
                    passwordVariable: 'GIT_TOKEN'
                )]) {
                    sh """
                        # Git config
                        git config user.email "jenkins@example.com"
                        git config user.name "Jenkins"
                        
                        # Tag banao
                        git tag -a ${VERSION} -m "Release ${VERSION}"
                        
                        # Tag push karo GitHub pe
                        git push https://${GIT_USER}:${GIT_TOKEN}@github.com/<username>/jenkins-versioning.git ${VERSION}
                    """
                }
            }
        }
        
        stage('Verify') {
            steps {
                sh """
                    echo "✅ Version: ${VERSION}"
                    echo "✅ Image pushed: ${IMAGE_NAME}:${VERSION}"
                    git tag -l
                """
            }
        }
    }
    
    post {
        success {
            echo "🎉 Build ${VERSION} Successfully Tagged!"
        }
        failure {
            echo "❌ Build Failed!"
        }
    }
}
